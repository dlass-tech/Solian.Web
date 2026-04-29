<script lang="ts">
	import { page } from '$app/stores';
	import { Puzzle } from 'lucide-svelte';
	import CaptchaWidget from '$lib/components/CaptchaWidget.svelte';

	function useToken(value: string) {
		const finalToken = value.trim();
		if (!finalToken) return;

		if (window.parent !== window) {
			window.parent.postMessage(`captcha_tk=${finalToken}`, '*');
		}

		const redirectUri = $page.url.searchParams.get('redirect_uri');
		if (redirectUri) {
			const url = new URL(redirectUri);
			url.searchParams.set('captcha_tk', finalToken);
			window.location.href = url.toString();
		}
	}
</script>

<svelte:head>
	<title>人机验证 - Dynamic Network</title>
</svelte:head>

<div
	class="mx-auto w-full max-w-lg rounded-3xl border border-base-300/70 bg-base-100/90 p-6 text-center shadow-2xl backdrop-blur-xl"
>
	<div class="mb-4 flex flex-col items-center gap-3">
		<div class="flex h-11 w-11 items-center justify-center rounded-2xl bg-primary/15 text-primary">
			<Puzzle size={20} />
		</div>
		<div>
			<h1 class="text-xl font-black">Dynamic Network 反机器人</h1>
			<p class="text-xs text-base-content/60">完成验证以继续</p>
		</div>
	</div>

	<div class="my-4">
		<CaptchaWidget onVerified={useToken} />
	</div>

	<p class="mt-5 text-xs text-base-content/60">
		支持方程序：
		<a
			href="https://github.com/dlass-tech/DysonNetwork"
			class="link link-primary"
			target="_blank"
			rel="noreferrer">DysonNetwork.Sphere</a
		>
	</p>
</div>
