<script lang="ts">
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';
	import { Button } from '@/components/ui/button';
	import * as Tabs from '@/components/ui/tabs';

	import { userProfile } from '@/stores/session.svelte';
	import admins from '@/data/admins';

	let value = $page.url.pathname;

	$: if ($page.url.pathname !== value) {
		goto(value);
	}

	async function signOut() {
		await $page.data.supabase.auth.signOut();
		goto('/');
	}

	$: email = $page.data.session?.user?.email;
	$: isAdmin = email && admins.includes(email);
</script>

<div class="flex items-center justify-evenly gap-2 border-b p-2 shadow-sm">
	{#if $page.data.session}
		<div class="grid gap-1 rounded-lg bg-muted px-2 py-1 text-sm text-muted-foreground">
			<span>
				{$userProfile?.username}
			</span>
		</div>
	{/if}

	<Tabs.Root bind:value>
		<Tabs.List>
			<Tabs.Trigger value="/owned">Owned</Tabs.Trigger>
			<Tabs.Trigger value="/">Buy</Tabs.Trigger>
			{#if $userProfile && $userProfile.is_vendor}
				<Tabs.Trigger value="/vendor">Sell</Tabs.Trigger>
			{/if}
			{#if isAdmin}
				<Tabs.Trigger value="/admin">Admin</Tabs.Trigger>
			{/if}
		</Tabs.List>
	</Tabs.Root>

	<!--  Sign out button -->
	<Button variant="destructive" icon="lucide:log-out" on:click={signOut}>Sign out</Button>
</div>
