<script>
    import { ethers } from "ethers";
    import contractAbi from '../contracts/QuizGame.json';
    let answer = null;
    let funding = null;
    export let web3Props = {
        provider: null,
        signer: null,
        account: null,
        chainId: null,
        contract: null
    };
    export let questionAddr = null;
    $: question = null;
    $: value = null;
    // funded will determine what CSS and functionality is available
    $: funded = value > 0 ? 'question-funded' : 'question-not-funded';
    let qContract = null;
    async function getQuestion() {
        // get the question contract
        qContract = new ethers.Contract(questionAddr, contractAbi.abi, web3Props.signer);
        // get the question
        question = await qContract.question();
        // get the value of a correct answer
        value = Number(ethers.utils.formatEther(await web3Props.provider.getBalance(questionAddr)));
        // listen for funding
        qContract.on('QuizFunded', (balance) => {
            console.log('QuizFunded', balance);
            value = Number(ethers.utils.formatEther(balance));
        });
        // listen for correct answers
        qContract.on('AnswerGuessed', () => {
            getQuestion();
        });
    }
    // submit a guess to the contract
    async function submitGuess() {
        await qContract.guess(answer);
    }
    // fund the quesiton
    async function fund() {
        web3Props.signer.sendTransaction({
            to: questionAddr,
            value: ethers.utils.parseEther(funding)
        });
        funding = null;
    }
    // run the getQuestion function
    getQuestion();
</script>

<div class="{funded} column width-10 offset-1 center">
    <div class="question">
        {question}
    </div>
    <div class="value">
        {value} ETH
    </div>
    <input type="text" bind:value={answer} />
    <button on:click={submitGuess} disabled={value <= 0}>Submit Answer</button>
    <br />
    <input type="text" bind:value={funding} />
    <button on:click={fund}>Fund</button>
</div>

<style>
    .question-funded {
        background: #4ee44e;
    }
    .question-not-funded {
        background: #ffb6c1;
    }
    .qwrap {
        overflow: hidden;
        position: relative;
        color: white;
        margin-bottom: 1rem;
        padding: 20px;
        border-radius: 15px;
        width: 50%;
        box-shadow: 1px 1px 4px rgba(0, 0, 0, 0.3);
    }
    .question {
        font-size: 2em;
    }
</style>