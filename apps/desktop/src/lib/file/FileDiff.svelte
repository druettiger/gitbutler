<script lang="ts">
	import HunkViewer from '$lib/hunk/HunkViewer.svelte';
	import InfoMessage from '$lib/shared/InfoMessage.svelte';
	import LargeDiffMessage from '$lib/shared/LargeDiffMessage.svelte';
	import { computeAddedRemovedByHunk } from '$lib/utils/metrics';
	import { getLocalCommits, getLocalAndRemoteCommits } from '$lib/vbranches/contexts';
	import { getLockText } from '$lib/vbranches/tooltip';
	import type { HunkSection, ContentSection } from '$lib/utils/fileSections';

	interface Props {
		filePath: string;
		isBinary: boolean;
		isLarge: boolean;
		sections: (HunkSection | ContentSection)[];
		isUnapplied: boolean;
		selectable: boolean;
		isFileLocked: boolean;
		readonly: boolean;
		commitId?: string;
	}

	let {
		filePath,
		isBinary,
		isLarge,
		sections,
		isUnapplied,
		commitId,
		selectable = false,
		isFileLocked = false,
		readonly = false
	}: Props = $props();

	let alwaysShow = $state(false);
	const localCommits = isFileLocked ? getLocalCommits() : undefined;
	const remoteCommits = isFileLocked ? getLocalAndRemoteCommits() : undefined;

	const commits = isFileLocked ? ($localCommits || []).concat($remoteCommits || []) : undefined;

	function getGutterMinWidth(max: number | undefined) {
		if (!max) {
			return 1;
		}
		if (max >= 10000) return 2.5;
		if (max >= 1000) return 2;
		if (max >= 100) return 1.5;
		if (max >= 10) return 1.25;
		return 1;
	}
	const maxLineNumber = $derived(sections.at(-1)?.maxLineNumber);
	const minWidth = $derived(getGutterMinWidth(maxLineNumber));
</script>

<div class="hunks">
	{#if isBinary}
		Binary content not shown
	{:else if isLarge}
		Diff too large to be shown
	{:else if sections.length > 50 && !alwaysShow}
		<LargeDiffMessage
			showFrame
			handleShow={() => {
				alwaysShow = true;
			}}
		/>
	{:else}
		{#each sections as section}
			{@const { added, removed } = computeAddedRemovedByHunk(section)}
			{#if 'hunk' in section}
				{@const isHunkLocked = section.hunk.lockedTo && section.hunk.lockedTo.length > 0 && commits}
				<div class="hunk-wrapper">
					{#if isHunkLocked || section.hunk.poisoned}
						<div class="indicators text-11 text-semibold">
							{#if isHunkLocked}
								<InfoMessage filled outlined={false} style="warning" icon="locked">
									<svelte:fragment slot="content"
										>{getLockText(section.hunk.lockedTo, commits)}</svelte:fragment
									>
								</InfoMessage>
							{/if}
							{#if section.hunk.poisoned}
								<InfoMessage filled outlined={false}>
									<svelte:fragment slot="content"
										>Can not manage this hunk because it depends on changes from multiple branches</svelte:fragment
									>
								</InfoMessage>
							{/if}
						</div>
					{/if}
					<HunkViewer
						{filePath}
						{section}
						{selectable}
						{isUnapplied}
						{isFileLocked}
						{minWidth}
						{readonly}
						{commitId}
						linesModified={added + removed}
					/>
				</div>
			{/if}
		{/each}
	{/if}
</div>

<style lang="postcss">
	.hunks {
		display: flex;
		flex-direction: column;
		position: relative;
		max-height: 100%;
		flex-shrink: 0;
		padding: 14px;
		gap: 16px;
	}
	.hunk-wrapper {
		display: flex;
		flex-direction: column;
		gap: 10px;
	}

	.indicators {
		display: flex;
		align-items: center;
		gap: 2px;
	}
</style>
