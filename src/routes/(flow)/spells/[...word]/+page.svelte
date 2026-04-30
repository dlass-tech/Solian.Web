<script lang="ts">
	import { page } from '$app/stores';
	import { onMount } from 'svelte';
	import { Wand2, User, Clock, Calendar, Check } from 'lucide-svelte';
	import { getSpell, applySpell, type SpellInfo } from '$lib/utils/api';

	const spellTypes = [
		'Account Activation',
		'Account Deactivation',
		'Account Deletion',
		'Reset Password',
		'Contact Method Verification'
	];

	let spell = $state<SpellInfo | null>(null);
	let error = $state<string | null>(null);
	let newPassword = $state('');
	let submitting = $state(false);
	let done = $state(false);
	let loading = $state(true);

	const spellWord = $derived($page.params.word ?? '');

	async function fetchSpell() {
		if (!spellWord) {
			error = 'Invalid spell word';
			loading = false;
			return;
		}
		loading = true;
		error = null;
		try {
			spell = await getSpell(spellWord);
		} catch (err) {
			error = err instanceof Error ? err.message : String(err);
		} finally {
			loading = false;
		}
	}

	async function apply() {
		if (!spell || submitting || !spellWord) return;
		submitting = true;
		error = null;
		try {
			await applySpell(spellWord, spell.type === 3 ? newPassword : undefined);
			done = true;
		} catch (err) {
			error = err instanceof Error ? err.message : String(err);
		} finally {
			submitting = false;
		}
	}

	onMount(fetchSpell);
</script>

<svelte:head>
	<title>魔术链接 - Dynamic Network</title>
</svelte:head>

<div class="w-full max-w-md">
	<div class="card/90 shadow-xl backdrop-blur">
		<div class="card-body">
			<div class="mb-4 flex items-center gap-3 border-b border-base-300 pb-4">
				<div class="rounded-xl bg-warning/20 p-2">
					<Wand2 class="text-warning" size={22} />
				</div>
				<div>
					<h1 class="text-xl font-bold">魔术链接</h1>
					<p class="text-sm text-base-content/60">
						{spell ? (spellTypes[spell.type] ?? 'Unknown Spell') : 'Loading...'}
					</p>
				</div>
			</div>

			{#if done}
				<div class="alert alert-success">
					<span>激活成功，你可以关掉这个页面了。</span>
				</div>
			{:else if error}
				<div class="alert alert-error">
					<span>{error}</span>
				</div>
			{:else if loading}
				<div class="flex justify-center py-10">
					<span class="loading loading-lg loading-spinner"></span>
				</div>
			{:else if spell}
				<div class="space-y-4 text-sm">
					<div class="flex items-center gap-3">
						<User size={18} class="text-base-content/60" />
						<div>
							<p class="text-xs tracking-wide text-base-content/60 uppercase">账户</p>
							<p class="font-semibold">@{spell.account.name}</p>
						</div>
					</div>

					<div class="flex items-center gap-3">
						<Clock size={18} class="text-base-content/60" />
						<div>
							<p class="text-xs tracking-wide text-base-content/60 uppercase">可用于</p>
							<p class="font-semibold">
								{new Date(spell.createdAt ?? spell.affectedAt).toLocaleString()}
							</p>
						</div>
					</div>

					{#if spell.expiredAt}
						<div class="flex items-center gap-3">
							<Calendar size={18} class="text-base-content/60" />
							<div>
								<p class="text-xs tracking-wide text-base-content/60 uppercase">过期</p>
								<p class="font-semibold">{new Date(spell.expiredAt).toLocaleString()}</p>
							</div>
						</div>
					{/if}
				</div>

				<div class="mt-5 space-y-3">
					{#if spell.type === 3}
						<input
							type="password"
							class="input-bordered input w-full"
							placeholder="新密码"
							bind:value={newPassword}
						/>
					{/if}
					<button
						class="btn w-full btn-primary"
						onclick={apply}
						disabled={submitting || (spell.type === 3 && newPassword.length === 0)}
					>
						{#if submitting}
							<span class="loading loading-sm loading-spinner"></span>
						{:else}
							<Check size={16} />
						{/if}
						激活账户
					</button>
				</div>
			{/if}
		</div>
	</div>
</div>
