<script lang="ts">
	import { markAsUnread } from '$lib/api/email';
	import { EmailFolder, type Email } from '$lib/types';
	import { get } from 'svelte/store';
	import EmailDetailView from './EmailDetailView.svelte';
	import ComposeEmail from './emailEditor/ComposeEmail.svelte';
	import { emailServiceState, user } from '$lib/store';
	import EmailCommandBar from './emailCommandBar/EmailCommandBar.svelte';
	import EmailList from '$lib/components/emailListContainer/EmailList.svelte';

	let { folder = EmailFolder.INBOX }: { folder: EmailFolder } = $props();
	let email = $state<Email | undefined>(undefined);
	let emailDetailView: EmailDetailView | undefined = undefined;
	let showComposeEmail = $state(get(emailServiceState).showComposeEmail);
	let composeEmailMode = $state(get(emailServiceState).composeEmailMode);

	emailServiceState.subscribe((state) => {
		if (showComposeEmail !== state.showComposeEmail) {
			showComposeEmail = state.showComposeEmail;
			composeEmailMode = state.composeEmailMode;
		}
		if (email !== state.currentEmail) {
			email = state.currentEmail;
		}
	});

	const selectEmail = (selectedEmail: Email) => {
		emailServiceState.update((state) => ({
			...state,
			currentEmail: selectedEmail
		}));
	};

	const markEmailAsUnread = (email: Email) => {
		const _markAsUnread = async () => {
			const res = await markAsUnread({ user: get(user)?.id.toString(), email_id: email.email_id });
			if (res) {
				emailServiceState.update((state) => ({
					...state,
					emailList: state.emailList.map((e) => (e.email_id === email.email_id ? res : e))
				}));
			}
		};
		_markAsUnread();
	};
</script>

<div class="email-container">
	<div class="email-section">
		<EmailCommandBar {markEmailAsUnread} {folder} />
		<div class="content">
			<EmailList {selectEmail} {folder} />

			<EmailDetailView bind:this={emailDetailView} />
			{#if showComposeEmail}
				{#key showComposeEmail}
					<ComposeEmail {email} {composeEmailMode} />
				{/key}
			{/if}
		</div>
	</div>
</div>

<style>
	.email-container {
		height: 100%;
		width: 100%;
		background-color: var(--color-primary-container);
		border-radius: 10px;
		padding-block: 4px;
		border: 1px solid var(--color-primary-dark-gray);
	}

	.email-section {
		display: flex;
		flex-direction: column;
		height: 100%;
		gap: 10px;
		width: 100%;
	}
	.content {
		display: flex;
		flex-direction: row;
		overflow: hidden;
		width: 100%;
		height: 100%;
	}
</style>
