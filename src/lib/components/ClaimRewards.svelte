<script>
    import WharfService, {eosBalance, rawRexBalance, rexBalance, unstakingBalances} from "$lib/wharf";
    import InfoRows from "$lib/components/InfoRows.svelte";
    import GlassBox from "$lib/components/GlassBox.svelte";
    import {commaNumber, readableNumber} from "$lib";
    import LoadingSpinner from "$lib/components/LoadingSpinner.svelte";


    $: withdrawable = ((_) => {

        return WharfService.withdrawableBalance();
    })($rawRexBalance);

    let loading = false;
    const claim = async () => {
        if (loading) return;
        loading = true;
        await WharfService.claim();
        loading = false;
    }

    $: unstaking = WharfService.pendingClaimableBalances().filter(x => !x.savings).map(x => {
        return [
            `${parseFloat(parseFloat(WharfService.convertRexToEos(parseFloat(x.rex.toString())).toString()).toFixed(4))} A`,
            x.date.toDateString()];
    }).sort((a, b) => +new Date(a[1]) - +new Date(b[1]));

    $: totalUnstaking = WharfService.pendingClaimableBalances().filter(x => !x.savings).reduce((acc, x) => parseFloat(parseFloat((acc + parseFloat(x.rex.toString())).toString()).toFixed(4)), 0);
    $: totalUnstakingInEos = parseFloat(parseFloat(WharfService.convertRexToEos(totalUnstaking).toString()).toFixed(4));
</script>

<GlassBox class="mt-2 overflow-hidden">

    {#if parseFloat(withdrawable) > 0}
        {#if !loading}
            <button class="btn" on:click={claim}>
                Claim Rewards
            </button>

            <InfoRows class="mt-2" rows={[
                ["You can claim", `${commaNumber(withdrawable)} A`, "font-black !text-yellow-300"]
            ]} />
        {:else}
            <button class="btn btn-disabled" disabled>
                <LoadingSpinner color="#fff" class="w-8 mx-auto" />
            </button>
        {/if}
    {:else}
        <figure class="btn btn-empty">
            Nothing to claim
        </figure>
    {/if}

    {#if unstaking.length}
        <section class="flex justify-between relative mt-10">
            <section>
                <figure class="text-sm">Currently unstaking</figure>
                <h1 class="text-3xl font-bold -mt-1">{totalUnstakingInEos} A</h1>
            </section>
        </section>
        <figure class="text-white text-xs text-opacity-50 mt-5">
            Below are your unstaking balances.
        </figure>
        <section class="inner-glass-box mt-2">
            <InfoRows rows={[
                ["Amount", "Date available", "!text-xs opacity-70"],
                ...unstaking
            ]} />
        </section>
    {/if}


</GlassBox>


<style lang="scss">
    .inner-glass-box {
        border-radius: 5px;
        box-shadow: 0 0 0 2px rgba(255,255,255, 0.2), 0 2px 4px rgba(0,0,0,0.4);
        background: rgba(255, 255, 255, 0.05);
        backdrop-filter: blur(6.599999904632568px);
        padding: 20px;
    }
</style>
