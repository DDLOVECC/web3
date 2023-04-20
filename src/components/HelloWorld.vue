<template>
    <div class="greetings">
        课堂练习
        <div>
            <button @click="connectWallet">Connect wallet</button>
        </div>
        <div>
            ChainID: {{chainIDs}}<br/>
            BlockNumber: {{blockNumber}}<br/>
            BlockTimestamp: {{timestamp}}<br/>
            Current Account: {{account}}<br/>
            Current Balance: {{balance}}<br/>
        </div>
        <div>
            <input v-model="contractAddress"/>
            <button @click="readContract">Read Contract</button>
            <br/>
            TokenSymbol: {{symbol}}<br/>
            TotalSupply: {{total}}<br/>
            Account Balance: {{balanceOf}}
        </div>

        <div>
            接收地址：<input v-model="toAddress"> 数量：<input v-model="amount">
            <button @click="transfer">transfer</button>
            <br/>
            estimateGas: {{estimateGas}}<br/>
            gasPrice: {{gasPrice}}<br/>
            txHash: {{txHash}}
        </div>
    </div>
    <div class="greetings">
        课后作业
        <div>
            增发数量：<input v-model="addAmount">
            <button @click="addAmountClick">增发</button>
        </div>
        <div>
            销毁数量：<input v-model="burnAmount">
            <button @click="burnAmountClick">销毁</button>
        </div>
        <div>
            totalSupply最新状态：{{newTotalSupply}}
            <button @click="findNewTotalSupplyClick">查看</button>
        </div>
        <div>
            查询地址：<input v-model="findAddress">
            <button @click="findNewBalanceClick">查询</button>
            余额：{{newBalance}}
        </div>
    </div>
</template>
<script>
    import Web3 from 'web3' ;

    export default {
        data() {
            return {
                contractAddress: "",
                amount: "",
                toAddress: "",
                chainIDs: "",
                blockNumber: "",
                timestamp: "",
                account: "",
                balance: "",
                abiJSON: [
                    {
                        "inputs": [
                            {
                                "internalType": "string",
                                "name": "_name",
                                "type": "string"
                            },
                            {
                                "internalType": "string",
                                "name": "_symbol",
                                "type": "string"
                            }
                        ],
                        "stateMutability": "nonpayable",
                        "type": "constructor"
                    },
                    {
                        "anonymous": false,
                        "inputs": [
                            {
                                "indexed": true,
                                "internalType": "address",
                                "name": "owner",
                                "type": "address"
                            },
                            {
                                "indexed": true,
                                "internalType": "address",
                                "name": "spender",
                                "type": "address"
                            },
                            {
                                "indexed": false,
                                "internalType": "uint256",
                                "name": "value",
                                "type": "uint256"
                            }
                        ],
                        "name": "Approval",
                        "type": "event"
                    },
                    {
                        "anonymous": false,
                        "inputs": [
                            {
                                "indexed": true,
                                "internalType": "address",
                                "name": "from",
                                "type": "address"
                            },
                            {
                                "indexed": true,
                                "internalType": "address",
                                "name": "to",
                                "type": "address"
                            },
                            {
                                "indexed": false,
                                "internalType": "uint256",
                                "name": "amount",
                                "type": "uint256"
                            }
                        ],
                        "name": "Fee",
                        "type": "event"
                    },
                    {
                        "anonymous": false,
                        "inputs": [
                            {
                                "indexed": true,
                                "internalType": "address",
                                "name": "from",
                                "type": "address"
                            },
                            {
                                "indexed": true,
                                "internalType": "address",
                                "name": "to",
                                "type": "address"
                            },
                            {
                                "indexed": false,
                                "internalType": "uint256",
                                "name": "value",
                                "type": "uint256"
                            }
                        ],
                        "name": "Transfer",
                        "type": "event"
                    },
                    {
                        "inputs": [
                            {
                                "internalType": "address",
                                "name": "owner",
                                "type": "address"
                            },
                            {
                                "internalType": "address",
                                "name": "spender",
                                "type": "address"
                            }
                        ],
                        "name": "allowance",
                        "outputs": [
                            {
                                "internalType": "uint256",
                                "name": "",
                                "type": "uint256"
                            }
                        ],
                        "stateMutability": "view",
                        "type": "function",
                        "constant": true
                    },
                    {
                        "inputs": [
                            {
                                "internalType": "address",
                                "name": "spender",
                                "type": "address"
                            },
                            {
                                "internalType": "uint256",
                                "name": "amount",
                                "type": "uint256"
                            }
                        ],
                        "name": "approve",
                        "outputs": [
                            {
                                "internalType": "bool",
                                "name": "",
                                "type": "bool"
                            }
                        ],
                        "stateMutability": "nonpayable",
                        "type": "function"
                    },
                    {
                        "inputs": [
                            {
                                "internalType": "address",
                                "name": "account",
                                "type": "address"
                            }
                        ],
                        "name": "balanceOf",
                        "outputs": [
                            {
                                "internalType": "uint256",
                                "name": "",
                                "type": "uint256"
                            }
                        ],
                        "stateMutability": "view",
                        "type": "function",
                        "constant": true
                    },
                    {
                        "inputs": [],
                        "name": "decimals",
                        "outputs": [
                            {
                                "internalType": "uint8",
                                "name": "",
                                "type": "uint8"
                            }
                        ],
                        "stateMutability": "view",
                        "type": "function",
                        "constant": true
                    },
                    {
                        "inputs": [
                            {
                                "internalType": "address",
                                "name": "spender",
                                "type": "address"
                            },
                            {
                                "internalType": "uint256",
                                "name": "subtractedValue",
                                "type": "uint256"
                            }
                        ],
                        "name": "decreaseAllowance",
                        "outputs": [
                            {
                                "internalType": "bool",
                                "name": "",
                                "type": "bool"
                            }
                        ],
                        "stateMutability": "nonpayable",
                        "type": "function"
                    },
                    {
                        "inputs": [],
                        "name": "feeConfigAddress",
                        "outputs": [
                            {
                                "internalType": "address",
                                "name": "",
                                "type": "address"
                            }
                        ],
                        "stateMutability": "view",
                        "type": "function",
                        "constant": true
                    },
                    {
                        "inputs": [
                            {
                                "internalType": "address",
                                "name": "spender",
                                "type": "address"
                            },
                            {
                                "internalType": "uint256",
                                "name": "addedValue",
                                "type": "uint256"
                            }
                        ],
                        "name": "increaseAllowance",
                        "outputs": [
                            {
                                "internalType": "bool",
                                "name": "",
                                "type": "bool"
                            }
                        ],
                        "stateMutability": "nonpayable",
                        "type": "function"
                    },
                    {
                        "inputs": [],
                        "name": "name",
                        "outputs": [
                            {
                                "internalType": "string",
                                "name": "",
                                "type": "string"
                            }
                        ],
                        "stateMutability": "view",
                        "type": "function",
                        "constant": true
                    },
                    {
                        "inputs": [],
                        "name": "owner",
                        "outputs": [
                            {
                                "internalType": "address",
                                "name": "",
                                "type": "address"
                            }
                        ],
                        "stateMutability": "view",
                        "type": "function",
                        "constant": true
                    },
                    {
                        "inputs": [],
                        "name": "symbol",
                        "outputs": [
                            {
                                "internalType": "string",
                                "name": "",
                                "type": "string"
                            }
                        ],
                        "stateMutability": "view",
                        "type": "function",
                        "constant": true
                    },
                    {
                        "inputs": [
                            {
                                "internalType": "address",
                                "name": "to",
                                "type": "address"
                            },
                            {
                                "internalType": "uint256",
                                "name": "amount",
                                "type": "uint256"
                            }
                        ],
                        "name": "transfer",
                        "outputs": [
                            {
                                "internalType": "bool",
                                "name": "",
                                "type": "bool"
                            }
                        ],
                        "stateMutability": "nonpayable",
                        "type": "function"
                    },
                    {
                        "inputs": [
                            {
                                "internalType": "address",
                                "name": "from",
                                "type": "address"
                            },
                            {
                                "internalType": "address",
                                "name": "to",
                                "type": "address"
                            },
                            {
                                "internalType": "uint256",
                                "name": "amount",
                                "type": "uint256"
                            }
                        ],
                        "name": "transferFrom",
                        "outputs": [
                            {
                                "internalType": "bool",
                                "name": "",
                                "type": "bool"
                            }
                        ],
                        "stateMutability": "nonpayable",
                        "type": "function"
                    },
                    {
                        "inputs": [],
                        "name": "totalSupply",
                        "outputs": [
                            {
                                "internalType": "uint256",
                                "name": "",
                                "type": "uint256"
                            }
                        ],
                        "stateMutability": "view",
                        "type": "function",
                        "constant": true
                    },
                    {
                        "inputs": [
                            {
                                "internalType": "uint256",
                                "name": "amount",
                                "type": "uint256"
                            }
                        ],
                        "name": "mint",
                        "outputs": [
                            {
                                "internalType": "bool",
                                "name": "",
                                "type": "bool"
                            }
                        ],
                        "stateMutability": "nonpayable",
                        "type": "function"
                    },
                    {
                        "inputs": [
                            {
                                "internalType": "uint256",
                                "name": "amount",
                                "type": "uint256"
                            }
                        ],
                        "name": "burn",
                        "outputs": [
                            {
                                "internalType": "bool",
                                "name": "",
                                "type": "bool"
                            }
                        ],
                        "stateMutability": "nonpayable",
                        "type": "function"
                    },
                    {
                        "inputs": [
                            {
                                "internalType": "address",
                                "name": "feeAddress",
                                "type": "address"
                            }
                        ],
                        "name": "configFeeAddress",
                        "outputs": [
                            {
                                "internalType": "bool",
                                "name": "",
                                "type": "bool"
                            }
                        ],
                        "stateMutability": "nonpayable",
                        "type": "function"
                    },
                    {
                        "inputs": [
                            {
                                "internalType": "address",
                                "name": "from",
                                "type": "address"
                            },
                            {
                                "internalType": "address",
                                "name": "to",
                                "type": "address"
                            },
                            {
                                "internalType": "uint256",
                                "name": "amount",
                                "type": "uint256"
                            },
                            {
                                "internalType": "uint256",
                                "name": "fee",
                                "type": "uint256"
                            }
                        ],
                        "name": "collectTransferFee",
                        "outputs": [
                            {
                                "internalType": "bool",
                                "name": "",
                                "type": "bool"
                            }
                        ],
                        "stateMutability": "nonpayable",
                        "type": "function"
                    },
                    {
                        "inputs": [
                            {
                                "internalType": "address",
                                "name": "recipient",
                                "type": "address"
                            },
                            {
                                "internalType": "uint256",
                                "name": "amount",
                                "type": "uint256"
                            }
                        ],
                        "name": "transferBurn",
                        "outputs": [
                            {
                                "internalType": "bool",
                                "name": "",
                                "type": "bool"
                            }
                        ],
                        "stateMutability": "nonpayable",
                        "type": "function"
                    }
                ],
                symbol: "",
                total: "",
                balanceOf: "",
                estimateGas: "",
                gasPrice: "",
                txHash: "",
                addAmount: "",
                burnAmount:"",
                findAddress: "",
                newTotalSupply: "",
                newBalance: ""
            }
        },
        methods: {
            async connectWallet() {
                if (window.ethereum) {
                    try {
                        await window.ethereum.enable();
                    } catch (e) {
                        console.error("用户取消链接")
                    }
                    const web3 = new Web3(window.ethereum);
                    const chainId = await web3.eth.getChainId();
                    const blockNumber = await web3.eth.getBlockNumber();
                    const block = await web3.eth.getBlock(blockNumber)
                    const account = await web3.eth.requestAccounts();
                    const balance = await web3.eth.getBalance(account[0]);
                    this.chainIDs = chainId;
                    this.blockNumber = blockNumber;
                    this.timestamp = block.timestamp;
                    this.account = account[0];
                    this.balance = balance;
                } else {
                    console.log("please install Metamask")
                }
            },
            async readContract() {
                const web3 = new Web3(window.ethereum);
                const contract = new web3.eth.Contract(this.abiJSON, this.contractAddress);
                const symbol = await contract.methods.symbol().call();
                const total = await contract.methods.totalSupply().call();
                const balanceOf = await contract.methods.balanceOf(this.account).call();
                this.symbol = symbol;
                this.total = total;
                this.balanceOf = balanceOf;
            },
            async transfer() {
                const web3 = new Web3(window.ethereum);
                const contract = new web3.eth.Contract(this.abiJSON, this.contractAddress);
                const toAddress = this.toAddress;
                const amount = this.amount;
                const transferData = await contract.methods.transfer(toAddress, web3.utils.toWei(amount)).encodeABI();
                const gasRes = await web3.eth.estimateGas({
                    to: this.contractAddress,
                    data: transferData,
                    from: this.account,
                    value: '0x0'
                });
                const gasPrice = await web3.eth.getGasPrice();
                const nonce = await web3.eth.getTransactionCount(this.account);
                const rawTransaction = {
                    from: this.account,
                    to: this.contractAddress,
                    nonce: web3.utils.toHex(nonce),
                    gasPrice: gasPrice,
                    gas: gasRes * 2,
                    value: '0x0',
                    data: transferData,
                    chainId: this.chainId
                }
                const tx = await web3.eth.sendTransaction(rawTransaction);
                console.log(tx.transactionHash)
                this.txHash = tx.transactionHash;
                this.estimateGas = gasRes;
                this.gasPrice = gasPrice;

            },
            async addAmountClick() {
                console.log("增发：{}",this.addAmount)
                const web3 = new Web3(window.ethereum);
                const contract = new web3.eth.Contract(this.abiJSON, this.contractAddress);
                const transferData = await contract.methods.mint(web3.utils.toWei(this.addAmount)).encodeABI();
                const gasRes = await web3.eth.estimateGas({
                    to: this.contractAddress,
                    data: transferData,
                    from: this.account,
                    value: '0x0'
                });
                const gasPrice = await web3.eth.getGasPrice();
                const nonce = await web3.eth.getTransactionCount(this.account);
                const rawTransaction = {
                    from: this.account,
                    to: this.contractAddress,
                    nonce: web3.utils.toHex(nonce),
                    gasPrice: gasPrice,
                    gas: gasRes * 2,
                    value: '0x0',
                    data: transferData,
                    chainId: this.chainId
                }
                const tx = await web3.eth.sendTransaction(rawTransaction);
                console.log(tx.transactionHash)
            },
            async burnAmountClick() {
                const web3 = new Web3(window.ethereum);
                const contract = new web3.eth.Contract(this.abiJSON, this.contractAddress);
                const transferData = await contract.methods.burn(web3.utils.toWei(this.burnAmount)).encodeABI();
                const gasRes = await web3.eth.estimateGas({
                    to: this.contractAddress,
                    data: transferData,
                    from: this.account,
                    value: '0x0'
                });
                const gasPrice = await web3.eth.getGasPrice();
                const nonce = await web3.eth.getTransactionCount(this.account);
                const rawTransaction = {
                    from: this.account,
                    to: this.contractAddress,
                    nonce: web3.utils.toHex(nonce),
                    gasPrice: gasPrice,
                    gas: gasRes * 2,
                    value: '0x0',
                    data: transferData,
                    chainId: this.chainId
                }
                const tx = await web3.eth.sendTransaction(rawTransaction);
                console.log(tx.transactionHash)
            },
            async findNewTotalSupplyClick() {
                const web3 = new Web3(window.ethereum);
                const contract = new web3.eth.Contract(this.abiJSON, this.contractAddress);
                const total = await contract.methods.totalSupply().call();
                this.newTotalSupply = total;
            },
            async findNewBalanceClick() {
                const web3 = new Web3(window.ethereum);
                const contract = new web3.eth.Contract(this.abiJSON, this.contractAddress);
                const balanceOf = await contract.methods.balanceOf(this.findAddress).call();
                this.newBalance = balanceOf;
            }

        }
    }
</script>
<style scoped>
    h1 {
        font-weight: 500;
        font-size: 2.6rem;
        top: -10px;
    }

    h3 {
        font-size: 1.2rem;
    }

    .greetings {
        border: 1px solid #2c3e50;
        padding: 0.5rem
    }

    .greetings h1,
    .greetings h3 {
        text-align: center;
    }

    @media (min-width: 1024px) {
        .greetings h1,
        .greetings h3 {
            text-align: center;
        }
    }
</style>
