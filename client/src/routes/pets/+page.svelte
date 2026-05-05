<script lang="ts">
	import { getPets } from '$lib/api/pet/PetController';
	import type { PetResponse } from '$lib/api/models';
	import { Input } from '$lib/components/ui/input';
	import * as Table from '$lib/components/ui/table';
	import { PawPrint, Search } from 'lucide-svelte';
	import { toast } from 'svelte-sonner';

	let pets = $state<PetResponse[]>([]);
	let loading = $state(true);
	let searchQuery = $state('');

	const filteredPets = $derived(() => {
		if (!searchQuery.trim()) return pets;
		const query = searchQuery.toLowerCase();
		return pets.filter((pet) => pet.name?.toLowerCase().includes(query));
	});

	async function loadPets() {
		loading = true;
		try {
			pets = await getPets();
		} catch (err) {
			toast.error('Failed to load pets');
			console.error('Error loading pets:', err);
		} finally {
			loading = false;
		}
	}

	function formatDate(dateStr: string | undefined): string {
		if (!dateStr) return 'Unknown';
		return new Date(dateStr).toLocaleDateString('en-US', {
			year: 'numeric',
			month: 'short',
			day: 'numeric'
		});
	}

	$effect(() => {
		loadPets();
	});
</script>

<svelte:head>
	<title>Pets | VetHub</title>
</svelte:head>

<div class="container mx-auto px-4 py-8">
	<!-- Header -->
	<div class="mb-8">
		<div class="flex items-center gap-3 mb-2">
			<div class="flex h-10 w-10 items-center justify-center rounded-lg bg-accent/10">
				<PawPrint class="h-5 w-5 text-accent" />
			</div>
			<h1 class="text-3xl font-bold text-foreground">Pets</h1>
		</div>
		<p class="text-muted-foreground">Manage pet patient records</p>
	</div>

	<!-- Search -->
	<div class="mb-6">
		<div class="relative max-w-md">
			<Search class="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-muted-foreground" />
			<Input
				type="search"
				placeholder="Search by pet name..."
				bind:value={searchQuery}
				class="pl-10"
			/>
		</div>
	</div>

	<!-- Table -->
	{#if loading}
		<div class="card p-12 text-center">
			<div class="mx-auto mb-4 h-8 w-8 animate-spin rounded-full border-4 border-primary border-t-transparent"></div>
			<p class="text-muted-foreground">Loading pets...</p>
		</div>
	{:else if filteredPets().length === 0}
		<div class="card p-12 text-center">
			<PawPrint class="mx-auto mb-4 h-12 w-12 text-muted-foreground/50" />
			{#if searchQuery}
				<p class="text-muted-foreground">No pets found matching "{searchQuery}"</p>
			{:else}
				<p class="text-muted-foreground">No pets registered yet</p>
			{/if}
		</div>
	{:else}
		<div class="card overflow-hidden">
			<Table.Root>
				<Table.Header>
					<Table.Row>
						<Table.Head>Name</Table.Head>
						<Table.Head>Type</Table.Head>
						<Table.Head>Birth Date</Table.Head>
						<Table.Head>Owner ID</Table.Head>
						<Table.Head>Visits</Table.Head>
						<Table.Head>ID</Table.Head>
					</Table.Row>
				</Table.Header>
				<Table.Body>
					{#each filteredPets() as pet (pet.id)}
						<Table.Row class="hover:bg-muted/50">
							<Table.Cell class="font-medium text-foreground">{pet.name}</Table.Cell>
							<Table.Cell>
								<span class="text-sm text-muted-foreground">{pet.type?.name ?? 'Unknown'}</span>
							</Table.Cell>
							<Table.Cell>
								<span class="text-sm text-muted-foreground">{formatDate(pet.birthDate)}</span>
							</Table.Cell>
							<Table.Cell>
								<span class="text-sm text-muted-foreground">{pet.ownerId ?? 'Unknown'}</span>
							</Table.Cell>
							<Table.Cell>
								<span class="text-sm text-muted-foreground">{pet.visits?.length ?? 0}</span>
							</Table.Cell>
							<Table.Cell>
								<span class="text-sm text-muted-foreground">{pet.id}</span>
							</Table.Cell>
						</Table.Row>
					{/each}
				</Table.Body>
			</Table.Root>
		</div>
		<p class="mt-4 text-sm text-muted-foreground">
			Showing {filteredPets().length} of {pets.length} pets
		</p>
	{/if}
</div>
