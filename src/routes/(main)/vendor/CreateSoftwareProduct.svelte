<script lang="ts">
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';
	import Button from '@/components/ui/button/button.svelte';
	import {
		DialogDescription,
		Dialog,
		DialogTrigger,
		DialogContent,
		DialogTitle,
		DialogFooter
	} from '@/components/ui/dialog';

	import Input from '@/components/ui/input/input.svelte';
	import Label from '@/components/ui/label/label.svelte';
	import Separator from '@/components/ui/separator/separator.svelte';
	import Textarea from '@/components/ui/textarea/textarea.svelte';
	import type { Database, Tables } from '@/types/supabase';
	import { useQueryClient } from '@sveltestack/svelte-query';
	import { toast } from 'svelte-sonner';

	let open = false;
	let loading = false;

	let form: Database['public']['Tables']['spps_software']['Insert'] = {
		vendor_id: $page.data.session?.user.id,
		title: '',
		description: '',
		package_name: ''
	};

	async function handleSubmit() {
		try {
			// toast.loading('Creating new software record');
			loading = true;
			const { error } = await $page.data.supabase.from('spps_software').insert(form);
			if (error) {
				toast.error(error.message ?? 'something went wrong');
				throw error;
			}
			toast.success('Created new software record');
			open = false;
			useQueryClient().invalidateQueries('vendor-software');
		} catch (error) {
			console.log(error);
		} finally {
			loading = false;
		}
	}
</script>

<Dialog bind:open>
	<DialogTrigger>
		<Button icon="lucide:plus">Add new software</Button>
	</DialogTrigger>
	<form on:submit|preventDefault={handleSubmit}>
		<DialogContent>
			<DialogTitle>Create new Software</DialogTitle>

			<DialogDescription>
				<p>
					New content is not published by default. Please visit the software page to publish it.
				</p>
			</DialogDescription>

			<Separator />

			<div class="flex flex-col gap-4 space-y-4">
				<div class="grid gap-2">
					<Label>Title</Label>
					<Input bind:value={form.title} required />
				</div>
				<div class="grid gap-2">
					<Label>Description</Label>
					<Textarea bind:value={form.description} />
				</div>
				<div class="grid gap-2">
					<Label>Package Name</Label>
					<Input bind:value={form.package_name} placeholder="com.google.maps ..." required />
				</div>
			</div>
			<Separator />

			<DialogFooter>
				<Button type="submit" on:click={handleSubmit} {loading}>Submit</Button>
			</DialogFooter>
		</DialogContent>
	</form>
</Dialog>
