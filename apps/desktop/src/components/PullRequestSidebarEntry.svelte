<script lang="ts">
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';
	import SidebarEntry from '$components/v3/SidebarEntry.svelte';
	import { Project } from '$lib/project/project';
	import { UserService } from '$lib/user/userService';
	import { parseDate } from '$lib/utils/time';
	import { getContext } from '@gitbutler/shared/context';
	import type { PullRequest } from '$lib/forge/interface/types';

	interface Props {
		pullRequest: PullRequest;
	}

	const { pullRequest }: Props = $props();

	const project = getContext(Project);

	const userService = getContext(UserService);
	const user = userService.user;

	const authorImgUrl = $derived.by(() => {
		return pullRequest.author?.email?.toLowerCase() === $user?.email?.toLowerCase()
			? $user?.picture
			: pullRequest.author?.gravatarUrl;
	});

	function onclick() {
		goto(formatPullRequestURL(project, pullRequest.number));
	}

	function formatPullRequestURL(project: Project, pullRequestNumber: number) {
		return `/${project.id}/pull/${pullRequestNumber}`;
	}

	const selected = $derived(
		$page.url.pathname === formatPullRequestURL(project, pullRequest.number)
	);
</script>

<SidebarEntry
	prTitle={pullRequest.title}
	remotes={[]}
	local={false}
	applied={false}
	lastCommitDetails={{
		authorName: pullRequest.author?.name || 'Unknown',
		lastCommitAt: parseDate(pullRequest.modifiedAt)
	}}
	pullRequestDetails={pullRequest && {
		title: pullRequest.title,
		draft: pullRequest.draft
	}}
	{onclick}
	{selected}
	avatars={[
		{
			name: pullRequest.author?.name || 'unknown',
			srcUrl: authorImgUrl || ''
		}
	]}
/>
