<script>
// @ts-nocheck

	import { slide } from 'svelte/transition'
    import { onMount } from 'svelte'
    import {
		accountChainId,
		connected,
		connectMetamask,
		connectWalletConnect,
		disconnect,
		walletAddress,
		accountProvider,
	} from '$lib/stores/provider';
	import { ethers } from "ethers";

    import { contractAddress, contractABI } from '../../lib/abis/contracts/contractABI'
    import { selectedOption, transactionID, awaitingTx, errorHandling } from '../../lib/stores/stores'
    
	import Dropdown from './other/Dropdown.svelte'
	import Guide  from './other/Needhelp.svelte'



    onMount(() => {
		provider = new ethers.providers.JsonRpcProvider(window['ethereum']);
        contractAddress.set(JSON.parse(localStorage.getItem('contractAddress')))
        
	})
    let provider
	let signer
	let contract

    let txid
    let txLog
    let txAwaiting
	let txSuccess  
    
    
    async function executeTransaction() {
        try {

            txLog = txid
            provider = new ethers.providers.Web3Provider(window['ethereum'])
            signer = provider.getSigner()
            contract = new ethers.Contract($contractAddress[0], contractABI, signer)

            if (txid == undefined || '') {
				console.log(txid)
				errorHandling.set('No transaction ID provided.');
			}
            const txExecute = await contract.execute(txid)
            errorHandling.set('')
            awaitingTx.set('Transaction executed. Awaiting transaction to be executed on network..')
            transactionID.set(txid)
            await txExecute.wait()
            //txSuccess = `<i style='text-green-500'>Transaction ID ${txLog} successfully executed.</i> <a class='text-blue-400' href='http://goerli.etherscan.io/tx/${txExecute.hash}' target='_blank'>Etherscan</a>`
            awaitingTx.set(`<i style='text-green-500'>Transaction ID ${txLog} successfully executed.</i> <a class='text-blue-400' href='http://goerli.etherscan.io/tx/${txExecute.hash}' target='_blank'>Etherscan</a>`)
        } catch (error) {
            if (error.message.includes('execution reverted: invalid owner')) {
				errorHandling.set('Transaction reverted: invalid owner')
			}
            if (error.message.includes('tx does not exist')) {
                errorHandling.set('Transaction reverted: Transaction ID does not exist.')
            }
            if (error.message.includes('approvals is less than required')) {
                errorHandling.set('Transaction reverted: Transaction ID approvals is less than required.')
            }
            if (error.message.includes('tx was already executed')) {
                errorHandling.set('Transaction reverted: This transaction ID was already executed.')
            }
        }
    }

</script>


<div>
    <div class="max-w-md w-full space-y-8 mt-2 ">
        <div class='mt-3'>
            {#if $errorHandling.includes('invalid owner')}
                <p class="text-center text-sm text-red-400" transition:slide>
                    {$errorHandling}
                </p>

            {:else if $errorHandling.includes('No transaction ID provided.')}
                <p class="text-center text-sm text-red-400" transition:slide>
                    {$errorHandling}
                </p>
            {:else if $errorHandling.includes('Transaction ID does not exist')}
                <p class="text-center text-sm text-red-400" transition:slide>
                    {$errorHandling}
                </p>
            {:else if $errorHandling.includes('Transaction ID approvals is less than required.')}
                <p class="text-center text-sm text-red-400" transition:slide>
                    {$errorHandling}
                </p>
            {:else if $errorHandling.includes('This transaction ID was already executed.')}
                <p class="text-center text-sm text-red-400" transition:slide>
                    {$errorHandling}
                </p>
            
            <!-- successful revoked transaction, process-->
            {:else if $awaitingTx.includes('Awaiting transaction to be executed on network..')}
                <p class="text-center text-sm text-grey dark:text-white" transition:slide>
                    <i>{$awaitingTx}</i>
                </p>
            {:else if $awaitingTx.includes('successfully executed.')}
                <p class="text-center text-sm dark:text-white" transition:slide>
                    {@html $awaitingTx}
                </p>
            {:else}
                <p class="text-center text-sm text-grey dark:text-white">
                    Execute a transaction (needs to be approved first)
                </p>
            {/if}
        </div>

        <div class="rounded-md shadow-sm -space-y-px mt-8">
            
            <div>
                <label for="txid" class="sr-only">txid</label>
                <input bind:value={txid} id="txidApprove" name="txid" type="number" min="0" step="1" required class="appearance-none rounded-none relative block w-full px-3 py-2 border border-gray-300 placeholder-gray-500 text-gray-900 rounded-b-md focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 focus:z-10 sm:text-sm" placeholder="Transaction ID">
            </div>
        </div>
      
        <div class='mb-1 w-full'>
            <button on:click={executeTransaction} type="submit" class="group relative w-full flex justify-center py-2 border border-transparent text-sm font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500">
                <span class="absolute left-0 inset-y-0 flex items-center pl-3">
                      <!-- Heroicon name: solid/lock-closed -->
                      <img src="https://upload.wikimedia.org/wikipedia/commons/8/82/Check_mark_9x9.svg" alt="approve icon" class="h-5 w-5">

                </span>
                Execute
                <span class="absolute right-0 pr-1">
                    <Dropdown></Dropdown>
                </span>
            </button>

            <div class="flex items-center justify-end mt-2"> <!-- Provides a prompt for a guide for specific function invocation.-->
                <button class='w-full'>
                    <div class="text-sm text-indigo-600 hover:text-indigo-400">
                        <Guide></Guide>
                    </div>
                </button>
            </div>
        </div>
    </div>
</div>