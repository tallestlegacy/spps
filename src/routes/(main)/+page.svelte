<script lang="ts">
	import { page } from '$app/stores';
	import Button from '@/components/ui/button/button.svelte';
	import { Card, CardContent, CardDescription, CardFooter, CardHeader } from '@/components/ui/card';
	import { useQuery, useQueryClient } from '@sveltestack/svelte-query';

	import { Skeleton } from '@/components/ui/skeleton';
	import type { Tables } from '@/types/supabase';
	import { toast } from 'svelte-sonner';
	import {
		AlertDialog,
		AlertDialogAction,
		AlertDialogCancel,
		AlertDialogContent,
		AlertDialogDescription,
		AlertDialogFooter,
		AlertDialogHeader,
		AlertDialogTitle,
		AlertDialogTrigger
	} from '@/components/ui/alert-dialog';

	import { generateToken } from '@/utils/strings';
	import { Avatar, AvatarImage, AvatarFallback } from '@/components/ui/avatar';

	const query = useQuery(
		'published-products',
		async () => {
			const { data, error } = await $page.data.supabase
				.from('spps_software')
				.select('*')
				.eq('published', true)
				.order('created_at', { ascending: false });

			if (error) throw error;
			return data;
		},
		{
			keepPreviousData: true
		}
	);

	const ownedQuery = useQuery<{ product_id: string }[], any>(
		'minimal-owned-products',
		async () => {
			const { data, error } = await $page.data.supabase
				.from('spps_receipts')
				.select('product_id')
				.eq('consumer_id', $page.data.session?.user.id);

			if (error) throw error;
			return data as { product_id: string }[];
		},
		{
			keepPreviousData: false,
			refetchOnMount: true
		}
	);

	function isOwned(product: Tables<'spps_software'>) {
		if (!$ownedQuery.data) {
			return 'Not Loaded';
		}

		return $ownedQuery.data.some((_product) => product.id === _product.product_id)
			? 'Owned'
			: 'Not Owned';
	}

	// purchase alert dialog variables
	let loading = false;
	let open = false;

	async function purchaseProduct(product: Tables<'spps_software'>) {
		loading = true;
		const { error } = await $page.data.supabase.from('spps_receipts').insert({
			product_id: product.id,
			consumer_id: $page.data.session?.user.id,
			token: generateToken()
		});

		// close dialog when the purchase is successfully
		loading = false;
		open = false;

		if (error) {
			toast.error(error.message);
		} else {
			//  refresh data on the owned and buy pages
			useQueryClient().invalidateQueries('minimal-owned-products');
			useQueryClient().invalidateQueries('owned-products');

			toast.success(`Purchased ${product.title} successfully!`);
		}
	}
</script>

<div class="grid grid-cols-[repeat(auto-fill,minmax(300px,1fr))] gap-4 overflow-auto p-4">
	{#if $query.isLoading}
		<!-- display skeletons while loading -->
		{#each { length: 3 } as _}
			<Card class="my-4">
				<CardContent>
					<Skeleton />
				</CardContent>
			</Card>
		{/each}
	{:else if $query.data}
		{#each $query.data as product}
			<Card class="flex flex-col">
				<!-- general sofwtare information -->
				<CardHeader>
					<div class="flex gap-4">
						<Avatar>
							<AvatarImage src={product.logo_url} alt={product.title} class="object-cover" />
							<AvatarFallback>{product.title.slice(0, 2)}</AvatarFallback>
						</Avatar>
						<p>{product.title}</p>
					</div>

					<div class="flex flex-col gap-2">
						<CardDescription class="flex items-center gap-2">
							<iconify-icon icon="lucide:package" />
							{product.package_name}
						</CardDescription>
						<CardDescription class="flex items-center gap-2">
							<iconify-icon icon="lucide:badge-dollar-sign" />
							{product.price} KES
						</CardDescription>
					</div>
				</CardHeader>

				<CardContent class="flex-1">
					<p class="text-sm">{product.description}</p>
				</CardContent>

				<!--   card footer contains a button to purchase new sofware if not owned yet -->
				<CardFooter class="flex items-center justify-between">
					<!--  re-render section when the query loading state changes -->
					{#key $ownedQuery.data}
						{#if $ownedQuery.data}
							<span class="rounded-full border px-3 py-1 text-xs">
								{isOwned(product)}
							</span>

							<!--  show an alert dialog to confirm product purchase -->
							<!-- remember to inform users that the sofwtare is still in testing so all purchases are mock -->

							{#if isOwned(product) === 'Not Owned'}
								<AlertDialog bind:open>
									<AlertDialogTrigger>
										<Button>Buy</Button>
									</AlertDialogTrigger>
									<AlertDialogContent>
										<AlertDialogHeader>
											<AlertDialogTitle>Purchase {product.price} KES</AlertDialogTitle>
											<AlertDialogDescription>
												Are you sure you want to purchase

												<b class="font-bold">
													{product.title}?
												</b>
											</AlertDialogDescription>
										</AlertDialogHeader>
										<AlertDialogFooter>
											<AlertDialogCancel>Cancel</AlertDialogCancel>
											<Button
												on:click={async () => {
													await purchaseProduct(product);
												}}
												{loading}>Purchase</Button
											>
										</AlertDialogFooter>
									</AlertDialogContent>
								</AlertDialog>
							{/if}
						{/if}
					{/key}
				</CardFooter>
			</Card>
		{/each}
	{/if}
</div>
