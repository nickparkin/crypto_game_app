<script> 
    import Question from '$lib/Question.svelte';
    export let web3Props = {provider: null, signer: null, account: null, chainId: null};
    $: questions = '';
    async function getQuestions(){
        questions = await web3Props.contract.getQuizzes();
        web3Props.contract.on('QuizCreated', (addr) => {
            getQuestions();

        });
    }
    getQuestions();
</script> 


{#if questions}
    <div class="column width-10 offset-1 center">
        {#each questions as questionAddr}
            <Question {questionAddr} {web3Props}/>
        {/each}
    </div>
{/if}
