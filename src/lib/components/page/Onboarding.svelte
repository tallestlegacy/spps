<script lang="ts">
	import { page } from '$app/stores';
	import { useQuery, useQueryClient } from '@sveltestack/svelte-query';

	import { user, userProfile } from '$lib/stores/session.svelte';

	import Input from '../ui/input/input.svelte';
	import type { Database } from '@/types/supabase';
	import Label from '../ui/label/label.svelte';
	import { Card, CardContent, CardFooter, CardHeader } from '../ui/card';
	import { toast } from 'svelte-sonner';
	import { Button } from '../ui/button';

	const query = useQuery('user_profile', async () => {
		const { data, error } = await $page.data.supabase
			.from('spps_user_profiles')
			.select('*')
			.eq('uid', $user?.id);

		if (error) throw error;

		userProfile.set(data[0]);
		return data;
	});

	const queryClient = useQueryClient();

	let form: Database['public']['Tables']['spps_user_profiles']['Insert'] = {
		username: '',
		uid: $user?.id as string,
		email: $user?.email as string
	};

	let loading = false;

	async function handleSubmit() {
		try {
			loading = true;
			const { error } = await $page.data.supabase.from('spps_user_profiles').insert(form);
			if (error) throw error;
			queryClient.invalidateQueries('user_profile');

			toast.success('Profile created');
		} catch (e) {
			toast.error(e.message);
		} finally {
			loading = false;
		}
	}
</script>

{#if $query.isLoading}
	<p>Loading ...</p>
{:else if $query.data?.length !== 0}
	<slot />
{:else}
	<!-- onboarding process -->

	<form
		on:submit|preventDefault={handleSubmit}
		class="flex h-screen items-center justify-center p-4"
	>
		<Card class="grid w-full max-w-screen-sm gap-4">
			<CardHeader class="text-lg font-bold">Your Identity</CardHeader>
			<CardContent>
				<div class="grid gap-4">
					<div class="grid gap-2">
						<Label>Email</Label>
						<Input bind:value={form.email} name="username" placeholder="Email" disabled />
					</div>
					<div class="grid gap-4">
						<Label>Username *</Label>
						<Input bind:value={form.username} name="username" placeholder="Username" />
					</div>
				</div>
			</CardContent>
			<CardFooter>
				<Button type="submit" class="w-full" disabled={!form.username} {loading}>Submit</Button>
			</CardFooter>
		</Card>
	</form>
{/if}
