<script lang="ts">
	import { page } from '$app/stores';
	import { PUBLIC_SUPABASE_URL } from '$env/static/public';
	import { Button } from '@/components/ui/button';
	import { Card, CardFooter, CardHeader } from '@/components/ui/card';
	import CardContent from '@/components/ui/card/card-content.svelte';
	import CardDescription from '@/components/ui/card/card-description.svelte';
	import { Input } from '@/components/ui/input';
	import { Label } from '@/components/ui/label';
	import Separator from '@/components/ui/separator/separator.svelte';
	import { Switch } from '@/components/ui/switch';
	import { Textarea } from '@/components/ui/textarea';
	import type { Database, Tables } from '@/types/supabase';
	import { useQuery, useQueryClient } from '@sveltestack/svelte-query';
	import dayjs from 'dayjs';
	import { toast } from 'svelte-sonner';

	let form: Tables<'spps_software'> = {
		background_url: null,
		created_at: '',
		description: null,
		file_url: null,
		id: '',
		logo_url: null,
		package_name: '',
		price: null,
		published: false,
		title: '',
		vender_id: ''
	};

	const query = useQuery<Tables<'spps_software'>, any>(
		['vendor-product', $page.params.product_id],
		async () => {
			const { data, error } = await $page.data.supabase
				.from('spps_software')
				.select('*')
				.eq('id', $page.params.product_id)
				.single();

			if (error) throw error;

			form = { ...data };

			return data;
		},
		{
			refetchOnMount: true
		}
	);

	let loading = false;

	async function handleUpdate() {
		try {
			loading = true;

			const { error } = await $page.data.supabase
				.from('spps_software')
				.update({ ...form, price: parseFloat(form.price) })
				.eq('id', $page.params.product_id)
				.single();

			if (error) throw error;

			toast.message(`Product record has been updated`);
			useQueryClient().invalidateQueries(['vendor-product', $page.params.product_id]);
		} finally {
			loading = false;
		}
	}

	let files: FileList;
	let fileLoading = false;

	async function handleFileUpload() {
		fileLoading = true;

		try {
			if (!files || files.length === 0) {
				return;
			}

			const file = files[0];

			const { data, error } = await $page.data.supabase.storage
				.from('spps')
				.upload(form.id, file, { upsert: true });

			toast.message('File uploaded');

			if (error) throw error;
			const filePath = `${PUBLIC_SUPABASE_URL}/storage/v1/object/public/spps/${data.path}`;

			await $page.data.supabase
				.from('spps_software')
				.update({ file_url: filePath })
				.eq('id', $page.params.product_id)
				.single();

			toast.message(`Product record has been updated`);
			useQueryClient().invalidateQueries(['vendor-product', $page.params.product_id]);
		} finally {
			fileLoading = false;
		}
	}

	const receiptsQuery = useQuery<Tables<'spps_receipts'>[], any>(
		['vendor-product-receipts', $page.params.product_id],
		async () => {
			const { data, error } = await $page.data.supabase
				.from('spps_receipts')
				.select('*')
				.eq('product_id', $page.params.product_id);

			if (error) throw error;
			return data;
		}
	);
</script>

<!-- header -->

<div class="flex h-screen flex-col overflow-hidden">
	<div class="flex h-fit items-center justify-between border-b px-8 py-4 text-xl font-bold">
		<!--  back arrow -->
		<a href="/vendor" class="flex items-center justify-center gap-2 text-sm italic text-primary">
			<iconify-icon icon="lucide:arrow-left" />
			<span> Back </span>
		</a>

		{#if $query.data}
			{$query.data.package_name}
		{/if}
	</div>

	<main class="overflow-auto">
		<div class="mx-auto flex h-full w-full max-w-screen-md flex-col gap-4 p-4">
			{#if $query.data}
				<div class="grid gap-2">
					<Label>Title</Label>
					<Input bind:value={form.title} />
				</div>
				<div class="grid gap-2">
					<Label>Package name</Label>
					<Input bind:value={form.package_name} disabled />
				</div>
				<div class="grid gap-2">
					<Label>Description</Label>
					<Textarea bind:value={form.description} />
				</div>
				<div class="grid gap-2">
					<Label>Price</Label>
					<Input bind:value={form.price} type="number" />
				</div>
				<div class="grid gap-2">
					<Label>Logo url</Label>
					<Input bind:value={form.logo_url} />
				</div>

				<div class="grid gap-2">
					<Label>Background url</Label>
					<Input bind:value={form.background_url} />
				</div>

				<!--  Publish toggle -->
				<div class="flex items-center space-x-2">
					<Switch bind:checked={form.published} id="published" />
					<Label for="published" class="cursor-pointer">Published</Label>
				</div>
				<Button {loading} on:click={handleUpdate}>Save</Button>

				<Separator />

				<Card>
					<CardHeader>
						<h3 class="text-xl">File upload</h3>
						<CardDescription>File must be less than 50MB size</CardDescription>
					</CardHeader>

					<CardContent>
						<input type="file" bind:files />
					</CardContent>

					<CardFooter class="grid gap-2">
						{#if files}
							<Button loading={fileLoading} on:click={handleFileUpload}>Upload</Button>
						{/if}

						{#if form.file_url}
							<span class="rounded-full border px-3 py-1 text-xs text-muted-foreground"
								>{form.file_url}</span
							>
						{/if}
					</CardFooter>
				</Card>
			{/if}
		</div>

		{#if $receiptsQuery.data}
			<Separator class="my-4" />

			<div class="grid gap-4 p-4">
				<h2 class="text-xl font-bold">Receipts</h2>

				<div class="grid grid-cols-[repeat(auto-fill,minmax(300px,1fr))] gap-2">
					{#each $receiptsQuery.data as receipt}
						<Card>
							<CardHeader>
								<div class="flex items-center gap-4">
									<iconify-icon icon="lucide:badge-check" />
									<h3 class="text-md font-medium">{receipt.token}</h3>
								</div>
								<CardDescription>
									Purchased on {dayjs(receipt.created_at).format('YYYY-MM-DD HH:mm')}
								</CardDescription>
							</CardHeader>
						</Card>
					{/each}
				</div>
			</div>
		{/if}
	</main>
</div>
