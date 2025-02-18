<script lang="ts">
	import { Button, Popup } from '../common'
	import { page } from '$app/stores'
	import { setQuery } from '$lib/navigation'
	import ToggleButton from '../common/toggleButton-v2/ToggleButton.svelte'
	import ToggleButtonGroup from '../common/toggleButton-v2/ToggleButtonGroup.svelte'
	import Tooltip from '../Tooltip.svelte'
	import { goto } from '$app/navigation'
	import AutoComplete from 'simple-svelte-autocomplete'
	import { ChevronDown, Filter, X } from 'lucide-svelte'
	import JsonEditor from '../apps/editor/settingsPanel/inputEditor/JsonEditor.svelte'

	export let paths: string[] = []
	export let selectedPath: string | undefined = undefined
	export let success: boolean | undefined = undefined
	export let isSkipped: boolean | undefined = undefined
	export let argFilter: string
	export let argError: string
	export let resultFilter: string
	export let resultError: string
	export let jobKindsCat: string
</script>

<div class="flex flex-col items-start gap-6 xl:gap-2 xl:flex-row mt-4 xl:mt-0">
	{#key selectedPath}
		<div class="relative">
			{#if selectedPath}
				<button
					class="absolute top-2 right-2 z-50"
					on:click={() => {
						selectedPath = undefined
						goto('/runs')
					}}
				>
					<X size={14} />
				</button>
			{:else}
				<ChevronDown class="absolute top-2 right-2" size={14} />
			{/if}

			<span class="text-xs absolute -top-4">Path</span>

			<AutoComplete
				items={paths}
				value={selectedPath}
				bind:selectedItem={selectedPath}
				inputClassName="!h-[30px] py-1 !text-xs !w-64"
				hideArrow
				className={selectedPath ? '!font-bold' : ''}
				dropdownClassName="!font-normal !w-64 !max-w-64"
			/>
		</div>
	{/key}
	<div class="flex flex-col xl:flex-row gap-6 xl:gap-2 w-full">
		<div class="relative">
			<span class="text-xs absolute -top-4">Kind</span>
			<ToggleButtonGroup
				bind:selected={jobKindsCat}
				on:selected={(e) => {
					const url = new URL($page.url)

					url.searchParams.set('job_kinds', e.detail)
					goto(url)
				}}
			>
				<ToggleButton value="all" label="All" />
				<ToggleButton value="runs" label="Runs" />
				<ToggleButton value="previews" label="Previews" />
				<ToggleButton value="dependencies" label="Dependencies" />
			</ToggleButtonGroup>
		</div>
		<div class="relative">
			<span class="text-xs absolute -top-4">Status</span>
			<ToggleButtonGroup
				bind:selected={success}
				on:selected={async () =>
					await setQuery($page.url, 'success', success === undefined ? success : String(success))}
			>
				<ToggleButton value={undefined} label="All" />
				<ToggleButton value={true} label="Success" class="whitespace-nowrap" />
				<ToggleButton value={false} label="Failure" class="whitespace-nowrap" />
			</ToggleButtonGroup>
		</div>
		<div class="relative">
			<span class="text-xs absolute -top-4">
				Flow
				<Tooltip light>Skipped flows are flows that did an early break</Tooltip></span
			>

			<ToggleButtonGroup
				bind:selected={isSkipped}
				on:selected={async () =>
					await setQuery(
						$page.url,
						'is_skipped',
						isSkipped === undefined ? isSkipped : String(isSkipped)
					)}
			>
				<ToggleButton value={undefined} label="All" class="whitespace-nowrap" />
				<ToggleButton value={false} label="Not skipped" class="whitespace-nowrap" />
				<ToggleButton value={true} label="Skipped" class="whitespace-nowrap" />
			</ToggleButtonGroup>
		</div>
	</div>

	<Popup
		floatingConfig={{ strategy: 'absolute', placement: 'bottom-end' }}
		containerClasses="border rounded-lg shadow-lg p-4 bg-surface"
		let:close
	>
		<svelte:fragment slot="button">
			<Button color="light" size="xs" nonCaptureEvent={true} variant="border">
				<div class="flex flex-row gap-1 items-center">
					<Filter size={16} />
					Filter by args/result
				</div>
			</Button>
		</svelte:fragment>
		<div class="flex flex-col w-72 p-2 gap-2">
			<span class="text-sm eading-6 font-semibold">Filters</span>
			<span class="text-xs leading-6">
				{`Filter by a json being a subset of the args/result. Try '\{"foo": "bar"\}'`}
			</span>
			<span class="text-xs eading-6 font-semibold">Filter by args</span>
			<JsonEditor on:change bind:error={argError} bind:code={argFilter} />
			<span class="text-xs eading-6 font-semibold">Filter by result</span>
			<JsonEditor on:change bind:error={resultError} bind:code={resultFilter} />

			<div class="flex flex-row gap-2 justify-between">
				<Button
					size="xs"
					color="light"
					on:click={() => {
						argFilter = ''
						resultFilter = ''
					}}
				>
					Clear
				</Button>
				<Button
					size="xs"
					color="dark"
					on:click={() => {
						close(null)
					}}
				>
					Close
				</Button>
			</div>
		</div>
	</Popup>
</div>
