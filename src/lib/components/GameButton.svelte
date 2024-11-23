<script>
	export let disableMode = false;
	export let text;
	export let onClick;
	export let compClass = "";
</script>

<button disabled={disableMode} on:click={onClick} class="{compClass} group bg-gray-700 shadow-lg cursor-pointer h-[80px] md:h-[101px] xl:h-[115px] min-w-[278px] rounded-xl max-h-[115px] disabled:cursor-not-allowed" type="button">
	<span class="rounded-xl select-none p-3 text-white bg-black min-w-[278px] max-h-[115px] h-full -translate-y-[14px] text-balance leading-tight text-[24px] md:text-4xl xl:text-5xl flex justify-center items-center group-active:-translate-y-[8px] group-disabled:shadow-[inset_0px_6px_12px_4px_rgba(255,255,255,0.5)] group-disabled:-translate-y-[6px]">
		{text ?? ""}
	</span>
</button>
