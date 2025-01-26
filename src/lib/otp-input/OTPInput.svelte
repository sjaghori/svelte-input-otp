<script lang="ts">
	import OTPItem from '../otp-item/OTPItem.svelte';

	type OTPInputProps = {
		numOfInputs?: number;
		value?: string;
		inputClassName?: string;
		containerClassName?: string;
		inputStyle?: string;
		containerStyle?: string;
		numberOnly?: boolean;
		placeholder?: string;
	};

	let {
		numOfInputs = 6,
		value = $bindable(''),
		inputClassName = '',
		containerClassName = '',
		inputStyle = '',
		containerStyle = '',
		numberOnly = false,
		placeholder = ''
	}: OTPInputProps = $props();

	let codes: string[] = $state([
		...value.slice(0, numOfInputs).split(''),
		...Array(numOfInputs <= value.length ? 0 : numOfInputs - value.length)
	]);

	let inputs: (null | HTMLInputElement)[] = $state(Array(numOfInputs).fill(null));

	$effect(() => {
		codes = [
			...value.slice(0, numOfInputs).split(''),
			...Array(numOfInputs <= value.length ? 0 : numOfInputs - value.length).fill('')
		];
	});

	$effect(() => {
		value = codes.join('');
	});

	let placeholders: string[] = $derived(
		placeholder.length < numOfInputs
			? [...placeholder.split(''), ...Array(numOfInputs - placeholder.length)]
			: placeholder.split('')
	);
</script>

<div class={containerClassName || 'wrapper'} style={containerStyle} role="group">
	{#each codes as _, index (index)}
		<OTPItem
			{index}
			{numberOnly}
			{inputs}
			{inputClassName}
			style={inputStyle}
			placeholder={placeholders[index]}
			bind:input={inputs[index]}
			bind:value={codes[index]}
			bind:codes
		/>
	{/each}
</div>

<style>
	.wrapper {
		display: flex;
		gap: 0.5rem;
		align-items: center;
	}
</style>
