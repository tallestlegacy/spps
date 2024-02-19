<script>
	import { useQuery, useQueryClient } from '@sveltestack/svelte-query';
	import CreateSoftwareProduct from './CreateSoftwareProduct.svelte';
	import { page } from '$app/stores';

	import { Card, CardContent, CardDescription, CardFooter, CardHeader } from '@/components/ui/card';
	import Skeleton from '@/components/ui/skeleton/skeleton.svelte';
	import Button from '@/components/ui/button/button.svelte';
	import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar';

	const query = useQuery(
		'vendor-software',
		async () => {
			const { data, error } = await $page.data.supabase
				.from('spps_software')
				.select('*, spps_receipts(id)')
				.order('title', { ascending: false })
				.eq('vendor_id', $page.data.session?.user.id);

			if (error) {
				console.log(error);
			}
			return data;
		},
		{
			keepPreviousData: true
		}
	);
</script>

<div class="overflow-auto p-4">
	<CreateSoftwareProduct />
	<div class="my-4 grid grid-cols-[repeat(auto-fill,minmax(300px,1fr))] gap-4">
		{#if $query.isLoading}
			{#each { length: 3 } as _}
				<Card>
					<CardHeader>
						<Skeleton class="h-20 w-full rounded-lg" />
					</CardHeader>

					<CardContent>
						<div class="flex flex-col gap-2">
							<Skeleton class="h-5 w-32 rounded-lg" />
							<Skeleton class="h-5 w-full rounded-lg" />
						</div>
					</CardContent>
				</Card>
			{/each}
		{:else if $query.data}
			{#each $query.data as software}
				<Card class="flex flex-col">
					<CardHeader class="flex flex-1 flex-col gap-1">
						<div class="flex gap-4">
							<Avatar>
								<AvatarImage src={software.logo_url} alt={software.title} class="object-cover" />
								<AvatarFallback>{software.title.slice(0, 2)}</AvatarFallback>
							</Avatar>
							<p>{software.title}</p>
						</div>

						<CardDescription class="flex items-center gap-2">
							<iconify-icon icon="lucide:package" />
							{software.package_name}</CardDescription
						>
						<CardDescription class="flex items-center gap-2">
							<iconify-icon icon="lucide:badge-dollar-sign" />
							{software.price}
						</CardDescription>
						<CardDescription class="flex items-center gap-2">
							<iconify-icon icon="lucide:download" />
							{software?.spps_receipts?.length ?? 0}
						</CardDescription>
					</CardHeader>

					<CardContent>
						<CardDescription>{software.description}</CardDescription>
					</CardContent>

					<CardFooter class="flex items-center justify-between">
						<span class="rounded-full border px-3 py-1 text-xs">
							{software.published ? 'Published' : 'Draft'}
						</span>
						<a href="/products/{software.id}">
							<Button variant="outline">View</Button>
						</a>
					</CardFooter>
				</Card>
			{/each}
		{/if}
	</div>
</div>
