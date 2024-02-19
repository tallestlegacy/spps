<script lang="ts">
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';
	import * as Tabs from '@/components/ui/tabs';

	let value = $page.url.pathname;

	$: if ($page.url.pathname !== value) {
		goto(value);
	}
</script>

<div class="flex items-center justify-center gap-2 border-b p-2 shadow-sm">
	<Tabs.Root bind:value>
		<Tabs.List>
			<Tabs.Trigger value="/owned">Owned</Tabs.Trigger>
			<Tabs.Trigger value="/">Buy</Tabs.Trigger>
			<Tabs.Trigger value="/vendor">Sell</Tabs.Trigger>
		</Tabs.List>
	</Tabs.Root>
</div>
