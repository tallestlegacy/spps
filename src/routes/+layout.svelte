<script>
	import '../app.css';
	import 'iconify-icon';
	import { ModeWatcher } from 'mode-watcher';

	import { invalidate } from '$app/navigation';
	import { onMount } from 'svelte';

	import { user } from '$lib/stores/session.svelte';

	import { Toaster } from '@/components/ui/sonner';

	export let data;

	let { supabase, session } = data;
	$: ({ supabase, session } = data);

	onMount(() => {
		const {
			data: { subscription }
		} = supabase.auth.onAuthStateChange((event, _session) => {
			user.set(_session?.user ?? null);

			if (_session?.expires_at !== session?.expires_at) {
				invalidate('supabase:auth');
			}
		});

		return () => subscription.unsubscribe();
	});

	// Svelte Stack
	import { QueryClient, QueryClientProvider } from '@sveltestack/svelte-query';
	const queryClient = new QueryClient({
		defaultOptions: {
			queries: {
				keepPreviousData: true,
				refetchOnWindowFocus: false,
				refetchOnMount: false
			}
		}
	});
</script>

<QueryClientProvider client={queryClient}>
	<!--  set shadcn to light mode by default -->
	<ModeWatcher defaultMode="light" />

	<!-- Sonner -->
	<Toaster />

	<!--  main content -->
	<slot />
</QueryClientProvider>
