<script lang="ts">
	import { Badge, Button } from '$lib/components/common'
	import { faPlus } from '@fortawesome/free-solid-svg-icons'
	import { Plus } from 'lucide-svelte'
	import { getContext } from 'svelte'
	import type { AppEditorContext } from '../../types'
	import { getAllScriptNames } from '../../utils'
	import PanelSection from '../settingsPanel/common/PanelSection.svelte'
	import { getAppScripts } from './utils'

	const PREFIX = 'script-selector-' as const

	export let selectedScriptComponentId: string | undefined = undefined
	const { app, selectedComponent, lazyGrid } = getContext<AppEditorContext>('AppEditorContext')
	let list: HTMLElement

	function selectInlineScript(id: string) {
		selectedScriptComponentId = id
		if (!id.startsWith('unused-') || !id.startsWith('bg_')) {
			$selectedComponent = selectedScriptComponentId
		}
	}

	$: runnables = getAppScripts($lazyGrid)

	// When selected component changes, update selectedScriptComponentId
	$: if (selectedComponent) {
		selectedScriptComponentId = $selectedComponent
	}

	// Doesn't work for some reason
	// $: if (selectedScriptComponentId) {
	// 	const selected = document.getElementById(PREFIX + selectedScriptComponentId)
	// 	if(selected) {
	// 		const top = selected.getBoundingClientRect().top - list.getBoundingClientRect().top + list.scrollTop
	// 		list.scrollTo({ top, behavior: 'smooth' })
	// 	}
	// }

	function createBackgroundScript() {
		let index = 0
		let newScriptPath = `Background Script ${index}`

		const names = getAllScriptNames($app)

		// Find a name that is not used by any other inline script
		while (names.includes(newScriptPath)) {
			newScriptPath = `Background Script ${++index}`
		}

		if (!$app.hiddenInlineScripts) {
			$app.hiddenInlineScripts = []
		}

		$app.hiddenInlineScripts.push({
			name: newScriptPath,
			inlineScript: undefined,
			fields: {}
		})
		$app.hiddenInlineScripts = $app.hiddenInlineScripts
		selectInlineScript(`bg_${$app.hiddenInlineScripts.length - 1}`)
	}
</script>

<div class="mb-0 pb-0 pl-2 sticky top-0 bg-white border-b text-xs font-semibold">Runnables</div>
<div bind:this={list} class="grow flex flex-col gap-4">
	<PanelSection title="Inline scripts" smallPadding>
		<div class="flex flex-col gap-2 w-full">
			{#if runnables.inline.length > 0}
				<div class="flex gap-1 flex-col ">
					{#each runnables.inline as { name, id }, index (index)}
						<button
							id={PREFIX + id}
							class="border flex gap-1 truncate font-normal justify-between w-full items-center py-1 px-2 rounded-sm duration-200 
							{selectedScriptComponentId === id ? 'border-blue-500 bg-blue-100' : 'hover:bg-blue-50'}"
							on:click={() => selectInlineScript(id)}
						>
							<span class="text-2xs truncate">{name}</span>
							<div>
								<Badge color="dark-indigo">{id}</Badge>
							</div>
						</button>
					{/each}
				</div>
			{/if}

			{#if $app.unusedInlineScripts?.length > 0}
				<div class="flex gap-1 flex-col ">
					{#each $app.unusedInlineScripts as unusedInlineScript, index (index)}
						{@const id = `unused-${index}`}
						<button
							id={PREFIX + id}
							class="border flex gap-1 truncate font-normal justify-between w-full items-center py-1 px-2 rounded-sm duration-200 
							{selectedScriptComponentId === id ? 'border-blue-500 bg-blue-100' : 'hover:bg-blue-50'}"
							on:click={() => selectInlineScript(id)}
						>
							<span class="text-2xs truncate">{unusedInlineScript.name}</span>
							<Badge color="red">Detached</Badge>
						</button>
					{/each}
				</div>
			{/if}

			{#if runnables.inline.length == 0 && $app.unusedInlineScripts?.length == 0}
				<div class="text-sm text-gray-500">No inline scripts</div>
			{/if}
		</div>
	</PanelSection>

	<PanelSection title="Workspace/Hub" smallPadding>
		<div class="flex flex-col gap-1 w-full">
			{#if runnables.imported.length > 0}
				{#each runnables.imported as { name, id }, index (index)}
					<button
						id={PREFIX + id}
						class="border flex gap-1 truncate font-normal justify-between w-full items-center py-1 px-2 rounded-sm duration-200 
							{selectedScriptComponentId === id ? 'border-blue-500 bg-blue-100' : 'hover:bg-blue-50'}"
						on:click={() => selectInlineScript(id)}
					>
						<span class="text-2xs truncate">{name}</span>
						<Badge color="dark-indigo">{id}</Badge>
					</button>
				{/each}
			{:else}
				<div class="text-sm text-gray-500">No imported scripts/flows</div>
			{/if}
		</div>
	</PanelSection>

	<PanelSection
		title="Background scripts"
		tooltip="Background scripts are triggered upon global refresh or when their input changes. The result
		of a background script can be shared among many components."
		smallPadding
	>
		<svelte:fragment slot="action">
			<Button size="xs" color="dark" variant="border" on:click={createBackgroundScript}
				><Plus size={14} />
			</Button>
		</svelte:fragment>
		<div class="flex flex-col gap-1 w-full">
			{#if $app.hiddenInlineScripts?.length > 0}
				{#each $app.hiddenInlineScripts as { name }, index (index)}
					{@const id = `bg_${index}`}
					<button
						id={PREFIX + id}
						class="border flex gap-1 truncate font-normal justify-between w-full items-center py-1 px-2 rounded-sm duration-200 
							{selectedScriptComponentId === id ? 'border-blue-500 bg-blue-100' : 'hover:bg-blue-50'}"
						on:click={() => selectInlineScript(id)}
					>
						<span class="text-2xs truncate">{name}</span>
						<Badge color="dark-indigo">{id}</Badge>
					</button>
				{/each}
			{:else}
				<div class="text-sm text-gray-500">No items</div>
			{/if}
		</div>
	</PanelSection>
</div>
