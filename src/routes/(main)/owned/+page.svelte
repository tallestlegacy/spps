<script lang="ts">
	import { page } from '$app/stores';
	import Button from '@/components/ui/button/button.svelte';
	import { Card, CardContent, CardDescription, CardFooter, CardHeader } from '@/components/ui/card';
	import { useQuery } from '@sveltestack/svelte-query';

	import { Skeleton } from '@/components/ui/skeleton';
	import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar';
	import { toast } from 'svelte-sonner';
	import type { Tables } from '@/types/supabase';

	const query = useQuery('owned-products', async () => {
		const { data, error } = await $page.data.supabase
			.from('spps_receipts')
			.select(
				`
          *,
          spps_software (*)
`
			)
			.eq('consumer_id', $page.data.session?.user.id)
			.order('created_at', { ascending: false });

		if (error) throw error;
		return data;
	});

	async function downloadFile(product: Tables<'spps_software'>) {
		toast.loading('Downloading...');
		const { data, error } = await $page.data.supabase.storage.from('spps').download(product.id);
		if (error) {
			toast.error(error.message);
		} else if (data) {
			toast.success('Downloaded Successfully');

			// download boilerplate code
			const url = window.URL.createObjectURL(data);
			const link = document.createElement('a');
			link.href = url;
			link.setAttribute('download', `${product.title}.spps`);
			document.body.appendChild(link);
			link.click();
			link.remove();
		}
	}
</script>

<div class="grid grid-cols-[repeat(auto-fill,minmax(300px,1fr))] gap-4 overflow-auto p-4">
	{#if $query.isLoading}
		{#each { length: 3 } as _}
			<Card class="my-4">
				<CardContent>
					<Skeleton />
				</CardContent>
			</Card>
		{/each}
	{:else if $query.data}
		{#if $query.data.length === 0}
			<p>You do not have any owned products yet</p>
		{:else}
			<!-- we'll do come fancy object destructuring to make the code more concise -->
			{#each $query.data as { token, spps_software: product }}
				<Card>
					<CardHeader>
						<div class="flex gap-4">
							<Avatar>
								<AvatarImage src={product.logo_url} alt={product.title} class="object-cover" />
								<AvatarFallback>{product.title.slice(0, 2)}</AvatarFallback>
							</Avatar>
							<p>{product.title}</p>
						</div>

						<CardDescription class="flex items-center gap-2">
							<iconify-icon icon="lucide:package" />
							{product.package_name}
						</CardDescription>
						<CardDescription class="flex items-center gap-2">
							<iconify-icon icon="lucide:badge-check" />
							{token}
						</CardDescription>
					</CardHeader>

					<CardFooter>
						<Button
							icon="lucide:download"
							on:click={() => {
								downloadFile(product);
							}}
						>
							Download
						</Button>
					</CardFooter>
				</Card>
			{/each}
		{/if}
	{/if}
</div>
