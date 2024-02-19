<script lang="ts">
	import { Button as ButtonPrimitive } from 'bits-ui';
	import { cn } from '$lib/utils';
	import { buttonVariants, type Props, type Events } from '.';

	type $$Props = Props;
	type $$Events = Events;

	let className: $$Props['class'] = undefined;
	export let variant: $$Props['variant'] = 'default';
	export let size: $$Props['size'] = 'default';
	export let builders: $$Props['builders'] = [];
	export let icon: $$Props['icon'] = undefined;
	export let loading = false;

	export { className as class };
</script>

<ButtonPrimitive.Root
	{builders}
	class={cn(buttonVariants({ variant, size, className }))}
	type="button"
	{...$$restProps}
	on:click
	on:keydown
>
	{#if icon}
		<iconify-icon {icon} class="mr-2" />
	{/if}

	<slot />

	{#if loading}
		<iconify-icon icon="svg-spinners:ring-resize" class="ml-2" />
	{/if}
</ButtonPrimitive.Root>
