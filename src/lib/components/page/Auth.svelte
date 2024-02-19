<script lang="ts">
	import Button from '$lib/components/ui/button/button.svelte';
	import CardContent from '../ui/card/card-content.svelte';
	import CardFooter from '../ui/card/card-footer.svelte';
	import CardHeader from '../ui/card/card-header.svelte';
	import Card from '../ui/card/card.svelte';
	import Separator from '$lib/components/ui/separator/separator.svelte';
	import { Github, Mail } from 'lucide-svelte';

	import type { Provider } from '@supabase/supabase-js';
	import { toast } from 'svelte-sonner';
	import { page } from '$app/stores';

	async function handleAuth(provider: Provider) {
		try {
			toast.message(`Attempting ${provider} login...`);
			// await sb.auth.signOut();
			const { error } = await $page.data.supabase.auth.signInWithOAuth({ provider });
			if (error) {
				toast.error(error?.message);
				console.log('Something went wrong');
				console.error(error);
			}
		} catch (error) {
			toast.error(error.toString());
		}
	}
</script>

<div
	class="flex h-screen items-center justify-center bg-gradient-to-bl from-cyan-400 to-emerald-500/20"
>
	<Card class="m-4 w-full max-w-screen-sm border-none shadow-xl">
		<CardHeader class="flex flex-col gap-2 space-y-0">
			<h2 class="text-xl font-bold"><b>SPPS</b> Authentication Portal</h2>
			<p class="text-sm">Please sign up with your preferred SSO method.</p>
		</CardHeader>

		<CardContent class="py-0">
			<div class="flex flex-col gap-2">
				<Button variant="secondary" class="flex gap-2" on:click={() => handleAuth('github')}>
					<Github size={16} /> Github
				</Button>
				<Button variant="secondary" class="flex gap-2">
					<Mail size={16} /> GMail
				</Button>
			</div>
		</CardContent>

		<Separator class="my-4" />

		<CardFooter class="space-y-0 text-sm text-muted-foreground">
			Signing up to the platform means you have agreed to our open policy. We do not collect any
			personal information, nor do we intend to in the future.
		</CardFooter>
	</Card>
</div>
