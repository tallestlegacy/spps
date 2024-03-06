<script lang="ts">
	import { page } from '$app/stores';

	import admins from '$lib/data/admins';

	$: email = $page.data.session?.user?.email;
	$: isAdmin = email && admins.includes(email);
</script>

{#if isAdmin}
	<slot />
{:else}
	<div class="flex h-full w-full flex-col items-center justify-center gap-4">
		<h1 class="text-4xl font-black">404 : Page not found</h1>
		<p>To go back to store, click <a href="/" class="text-pink-600 underline">here</a></p>
	</div>
{/if}
