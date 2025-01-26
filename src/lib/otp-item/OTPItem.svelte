<script lang="ts">
	type OTPItemProps = {
		input?: null | HTMLInputElement;
		index: number;
		value: string;
		codes: string[];
		inputs: (null | HTMLInputElement)[];
		inputClassName: string;
		numberOnly: boolean;
		style: string;
		placeholder: string;
	};

	let {
		input = $bindable(null),
		index,
		value = $bindable(),
		codes = $bindable(),
		inputs,
		inputClassName: className,
		numberOnly,
		style,
		placeholder
	}: OTPItemProps = $props();

	let key: string;

	const shiftFocus = (key: string) => {
		if (
			(!/[0-9]/.test(key) && numberOnly && key) ||
			key === 'ArrowRight' ||
			key === 'ArrowLeft' ||
			key === 'Backspace'
		)
			return;
		if (value === ' ') {
			value = '';
			return;
		}
		if (index !== inputs.length - 1) inputs[index + 1]?.focus();
	};

	const keyDownHandler = (e: KeyboardEvent) => {
		if (e.ctrlKey && e.key === 'z') {
			e.preventDefault();
		}
		key = e.key;
		if (value.length >= 1 && !e.ctrlKey) shiftFocus(key);
	};

	const typeHandler = (e: KeyboardEvent) => {
		if (value.length >= 1 || (!/[0-9]/.test(e.key) && numberOnly)) {
			e.preventDefault();
		}
	};

	const changeHandler = (e: Event) => {
		const val = (e.target as HTMLInputElement).value;
		if (/[0-9]/.test(val) || !numberOnly || !val) {
			codes = codes.map((c, i) => {
				if (i < index) {
					return c === '' ? ' ' : c;
				} else if (i === index) {
					return val[0];
				}
				return c;
			});

			if (val.length > 1) {
				for (let i = 0; i < val.length; i++) {
					if (inputs && inputs[index + i]) {
						inputs[index + i]!.value = val[i];
						codes[index + i] = val[i];
					}
				}

				focusOnLastFilledInput();

				return;
			}

			if (!val) {
				const len = codes.length;
				const filtered = codes.filter((_, i) => i !== index);
				codes = [...filtered, ...Array(len - filtered.length).fill('')];
			}
			shiftFocus(key);
		}
	};

	const keyUpHandler = (e: KeyboardEvent) => {
		if ((e.key === 'Backspace' || e.key === 'ArrowLeft') && index !== 0) {
			inputs[index - 1]?.focus();
		} else if (e.key === 'ArrowRight' && index !== inputs.length - 1) {
			inputs[index + 1]?.focus();
		}
	};

	const pasteHandler = (e: ClipboardEvent) => {
		e.preventDefault();
		const paste = e.clipboardData?.getData('text');
		if (!paste) return;
		let pasteValue = paste.replace(numberOnly ? /[^0-9]/g : '', '').slice(0, codes.length - index);
		const newCodes = [
			...codes.slice(0, index),
			...pasteValue.split(''),
			...codes.slice(index + pasteValue.length)
		];
		codes = newCodes;

		focusOnLastFilledInput();
	};

	const focusOnLastFilledInput = () => {
		const lastFilledIndex = codes.findIndex((c) => c === '') - 1;
		if (lastFilledIndex >= 0) {
			inputs[lastFilledIndex]?.focus();
		}
	};
</script>

<input
	class={className || 'default-input'}
	pattern={numberOnly ? '[0-9]*' : undefined}
	autocomplete={index === 0 ? 'one-time-code' : undefined}
	maxlength={index !== 0 ? 1 : undefined}
	inputmode={index === 0 ? 'numeric' : undefined}
	{style}
	{value}
	{placeholder}
	onkeydown={keyDownHandler}
	onkeyup={keyUpHandler}
	onkeypress={typeHandler}
	oninput={changeHandler}
	onpaste={pasteHandler}
	bind:this={input}
/>

<style>
	.default-input {
		width: 30px;
		height: 40px;
		text-align: center;
		border: 1px solid black;
		margin: 0;
	}
</style>
