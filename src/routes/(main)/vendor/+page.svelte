<script>
	import { useQuery, useQueryClient } from '@sveltestack/svelte-query';
	import CreateSoftwareProduct from './CreateSoftwareProduct.svelte';
	import { page } from '$app/stores';

	import { Card, CardContent, CardDescription, CardFooter, CardHeader } from '@/components/ui/card';
	import Skeleton from '@/components/ui/skeleton/skeleton.svelte';
	import Button from '@/components/ui/button/button.svelte';

	const query = useQuery(
		'vendor-software',
		async () => {
			const { data, error } = await $page.data.supabase
				.from('spps_software')
				.select('*')
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
					<CardHeader class="flex-1">
						<p>{software.title}</p>
						<CardDescription>{software.package_name}</CardDescription>
						<CardDescription>{software.description}</CardDescription>
					</CardHeader>

					<CardFooter class="flex items-center justify-between">
						<span class="rounded-full border px-3 py-1 text-xs">
							{software.published ? 'Published' : 'Draft'}
						</span>
						<a href="/products/{software.id}">
							<Button variant="outline">Edit</Button>
						</a>
					</CardFooter>
				</Card>
			{/each}
		{/if}
	</div>
</div>
