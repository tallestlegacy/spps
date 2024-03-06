<script lang="ts">
	import Auth from '@/components/page/Auth.svelte';
	import { user } from '@/stores/session.svelte';
	import MainNavbar from './MainNavbar.svelte';
	import Onboarding from '@/components/page/Onboarding.svelte';

	export let data;
</script>

{#if $user}
	<div class="flex h-screen flex-col overflow-y-auto">
		<Onboarding>
			<MainNavbar />
			<slot />
		</Onboarding>
	</div>
{:else}
	<Auth />
{/if}
