<script lang="ts">
	import { page } from '$app/stores';
	import { Card, CardContent, CardDescription, CardHeader } from '@/components/ui/card';
	import { Switch } from '@/components/ui/switch';
	import { useQuery, useQueryClient } from '@sveltestack/svelte-query';
	import { toast } from 'svelte-sonner';

	const usersQuery = useQuery('users', async () => {
		const { data, error } = await $page.data.supabase
			.from('spps_user_profiles')
			.select('*')
			.order('username', { ascending: false });
		if (error) throw error;
		return data;
	});

	const queryClient = useQueryClient();

	async function markAsVendor(uid: string, username: string, value: boolean) {
		toast.loading('Saving changes ...');
		const { error } = await $page.data.supabase
			.from('spps_user_profiles')
			.update({ is_vendor: value })
			.eq('uid', uid);
		if (error) toast.error(error.message);
		else {
			toast.success(`${username} has been marked as a vendor`);
			queryClient.invalidateQueries('users');
		}
	}
</script>

{#if $usersQuery.data}
	<div class="grid grid-cols-[repeat(auto-fill,minmax(300px,1fr))] gap-4 p-4">
		{#each $usersQuery.data as user}
			<Card>
				<CardHeader>
					<h4 class="text-md font-semibold">
						{user.username}
					</h4>
					<span class="text-sm text-muted-foreground">
						{user.email}
					</span>
				</CardHeader>
				<CardContent>
					<CardDescription class="flex flex-col gap-4">
						<!-- mark user as a vendor -->
						<div class="flex gap-4">
							Marked as vendor
							<Switch
								checked={user.is_vendor}
								onCheckedChange={() => markAsVendor(user.uid, user.username, !user.is_vendor)}
							/>
						</div>
					</CardDescription>
				</CardContent>
			</Card>
		{/each}
	</div>
{/if}
