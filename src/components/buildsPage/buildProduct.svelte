<script lang="ts">
	// @ts-ignore
	import Link from "$components/Link.svelte";
	import Chip from "$components/Chip.svelte";
	import IntersectionObserver from "svelte-intersection-observer";
	import { clickOutside } from '$lib/clickOut.js';
	import { fade, fly } from "svelte/transition";
	import { parts } from "$lib/stores/buildsStore";
	import type { Module } from "$lib/types/module";
	import { addPart } from "$lib/addPart";
	import { page } from "$app/stores";
	import { spring } from "svelte/motion";

	const url = $page.url.pathname;

	let element: HTMLElement;
	let intersecting: boolean;

	export let color: string;
	export let title: string;
	export let price: string;
	export let point1 = "";
	export let point2 = "";
	export let point3 = "";
	export let point4 = "";
	export let point5 = "";
	export let info: string = "";
	export let text: string;
	export let href: string;
	export let img: string;
	export let category: string = "";

	let open = false;
		let zoomed = false;

	function handleClickOutside() {
		open = false;
	}

	let x = spring(0);
	let y = spring(0);
	function zoom(event: MouseEvent) {
		if (zoomed) {
			var zoomer = event.currentTarget;
			// console.log(zoomer)

			let offsetX, offsetY;

			event.offsetX ? offsetX = event.offsetX : offsetX = event.touches[0].pageX
			event.offsetY ? offsetY = event.offsetY : offsetY = event.touches[0].pageY
			$x = offsetX/zoomer.offsetWidth*100
			$y = offsetY/zoomer.offsetHeight*100
			zoomer.style.transformOrigin = $x + '% ' + $y + '%';
		}
	}

	function keydown(event: KeyboardEvent) {
		if (event.key === "Escape") {
			open = false;
		}

		if (event.key === "Shift") {
			zoomed = true
		}
	}

	function keyup(event: KeyboardEvent) {
		if (event.key === "Shift") {
			zoomed = false
		}
	}

	let infoObjects: { text: string; tooltip: string }[] = [];
	if (info) {
		const infoArray = info.split(";");
		infoObjects = infoArray.map((item) => {
			const split = item.split("<");
			if (split.length === 1) {
				return { text: split[0], tooltip: "" };
			} else {
				return { text: split[0], tooltip: split[1].slice(0, -1) };
			}
		});
		price = infoObjects[0].text;
	}
</script>

<svelte:window on:keydown={keydown} on:keyup={keyup}/>

{#if open}
	<div in:fade={{duration: 300, delay: 0}} out:fade={{delay: 200}} class="fixed w-screen h-screen top-0 left-0 z-[60] backdrop-blur-sm md:flex hidden items-center justify-center bg-black bg-opacity-80">
		<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
		<div 
			transition:fly={{duration: 300, y: 200, delay: 100}} 
			class="h-full aspect-auto flex flex-col py-8 justify-center z-10 select-none relative" 
			use:clickOutside 
			on:clickOutside={handleClickOutside}
		>
			<img 
				class="select-none rounded-2xl object-contain h-full duration-150"
				style="scale: {zoomed ? "300%" : "100%"}; transition-property: scale"
				src={img}
				alt={title + " image"}
				crossorigin="anonymous"
				on:mousemove={(event) => zoom(event)}
			>
			{#if !zoomed}
				<button 
					on:click={() => open = false}
					transition:fade={{duration: 50, delay: 0}}
					class="stroke-red duration-300 absolute top-8 right-0 rounded-full border-2 border-red/40 bg-red/20 hover:bg-red/40">
					<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" class="w-7 h-7 rotate-45">
						<path stroke-linecap="round" stroke-linejoin="round" d="M12 4.5v15m7.5-7.5h-15" />
					</svg>
				</button>
			{/if}
		</div>
		{#if !zoomed}
			<div class="absolute bottom-1 left-1 text-sm text-neutral-400/60 md:visible invisible">Press Shift to zoom</div>
		{/if}
	</div>
{/if}

<IntersectionObserver {element} bind:intersecting>
	<div class="product-box h-full flex flex-col not-intersecting" bind:this={element} class:intersecting={intersecting}>
		<div class="{color} {category} relative h-fit w-full border-l-4 product pl-2 my-4 md:mr-8">
			<div class="flex w-full justify-between link t-{color}">
				<div>
					<Link {color} color1={color} size="2" {href} external={true}>{title}</Link>
				</div>
				<div>
					<button 
						on:click={() => addPart(title, price, color, category, url, href)}
						class="mr-2 hover:stroke-current stroke-contrast-500 duration-300">
						<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" class="w-7 h-7">
							<path stroke-linecap="round" stroke-linejoin="round" d="M12 4.5v15m7.5-7.5h-15" />
						</svg>
					</button>
					{#if img}
						<button on:click={() => open = true} class=" hover:stroke-current stroke-contrast-500 duration-300">
							<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke-width="2" class="w-7 h-7">
								<path stroke-linecap="round" stroke-linejoin="round" d="M2.25 15.75l5.159-5.159a2.25 2.25 0 013.182 0l5.159 5.159m-1.5-1.5l1.409-1.409a2.25 2.25 0 013.182 0l2.909 2.909m-18 3.75h16.5a1.5 1.5 0 001.5-1.5V6a1.5 1.5 0 00-1.5-1.5H3.75A1.5 1.5 0 002.25 6v12a1.5 1.5 0 001.5 1.5zm10.5-11.25h.008v.008h-.008V8.25zm.375 0a.375.375 0 11-.75 0 .375.375 0 01.75 0z" />
							</svg>
						</button>
					{/if}
				</div>
			</div>
			{#if info === ""}
				<div class="flex justify-between mt-3 mb-3 flex-wrap tracking-[0.01em]">
					<div class="text-base text-main-100 dark:text-contrast-300 mr-2">{price}</div>
					<div class="text-base text-main-50 dark:text-contrast-500 mr-2">{point1}</div>
					<div class="text-base text-main-50 dark:text-contrast-500 mr-2">{point2}</div>
					<div class="text-base text-main-50 dark:text-contrast-500 mr-2">{point3}</div>
					<div class="text-base text-main-50 dark:text-contrast-500 mr-2">{point4}</div>
					<div class="text-base text-main-50 dark:text-contrast-500 mr-2">{point5}</div>
				</div>
			{:else}
				<div class="relative flex flex-wrap w-fit gap-2 chip-box my-2">
					{#each infoObjects as infoObject}
						<Chip classes="text-sm" color={color} tooltip={infoObject.tooltip}>{infoObject.text}</Chip>
					{/each}
				</div>
			{/if}
			{#if text !== undefined}
				<div>{@html text}</div>
			{/if}
		</div>
	</div>
</IntersectionObserver>


<style>
	.not-intersecting {
		opacity: 0;
		transform: translateX(-5%);
		/* filter: blur(2px); */
		transition: all 300ms;
	}
	.intersecting {
		opacity: 1;
		transform: translateX(0%);
		/* filter: blur(0px); */
	}

	@media (min-width: 2000px) {
		.product-box:nth-of-type(4n+1) {
			transition-delay: 0ms;
		}

		.product-box:nth-of-type(4n+2) {
			transition-delay: 50ms;
		}

		.product-box:nth-of-type(4n+3) {
			transition-delay: 100ms;
		}

		.product-box:nth-of-type(4n+4) {
			transition-delay: 150ms;
		}
	}
	
	@media (min-width: 1280px) and (max-width: 1999px) {
		.product-box:nth-of-type(3n+1) {
			transition-delay: 0ms;
		}

		.product-box:nth-of-type(3n+2) {
			transition-delay: 50ms;
		}

		.product-box:nth-of-type(3n+3) {
			transition-delay: 100ms;
		}
	}

	@media (min-width: 768px) and (max-width: 1279px) {
		.product-box:nth-of-type(2n+1) {
			transition-delay: 0ms;
		}

		.product-box:nth-of-type(2n+2) {
			transition-delay: 50ms;
		}
	}

	@media (max-width: 767px) {
		.product-box:nth-of-type(1n+1) {
			transition-delay: 0ms;
		}
	}
</style>