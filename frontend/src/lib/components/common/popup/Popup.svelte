<script lang="ts">
	import { Popover, PopoverButton, PopoverPanel, Transition } from '@rgossiaux/svelte-headlessui'
	import Portal from 'svelte-portal'
	import { createFloatingActions, type ComputeConfig } from 'svelte-floating-ui'

	export let floatingConfig: ComputeConfig = {
		strategy: 'absolute',
		placement: 'bottom-start'
	}

	export let containerClasses: string = 'rounded-lg shadow-lg p-4 bg-surface'

	const [floatingRef, floatingContent] = createFloatingActions(floatingConfig)
</script>

<Popover let:open on:close>
	<PopoverButton>
		<div use:floatingRef>
			<slot name="button" />
		</div>
	</PopoverButton>
	<Portal>
		<div use:floatingContent class="z5000" hidden={!open}>
			<Transition
				enter="transition ease-out duration-200"
				enterFrom="opacity-0 translate-y-1"
				enterTo="opacity-100 translate-y-0"
				leave="transition ease-in duration-150"
				leaveFrom="opacity-100 translate-y-0"
				leaveTo="opacity-0 translate-y-1"
			>
				<PopoverPanel let:close>
					<div class={containerClasses}>
						<slot {close} />
					</div>
				</PopoverPanel>
			</Transition>
		</div>
	</Portal>
</Popover>
