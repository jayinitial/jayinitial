const WebSocket = require('ws');
const fs = require('fs');
const solanaWeb3 = require('@solana/web3.js');
const {Connection, clusterApiUrl, PublicKey, Keypair, TransactionMessage, VersionedTransaction, SystemProgram, TransactionInstruction, LAMPORTS_PER_SOL, ComputeBudgetProgram} = require("@solana/web3.js");
const splToken = require('@solana/spl-token');
const {createAssociatedTokenAccountIdempotentInstruction, createCloseAccountInstruction, createAssociatedTokenAccountInstruction, getAssociatedTokenAddressSync} = require('@solana/spl-token')
const {TOKEN_PROGRAM_ID, AccountLayout, createMint, getOrCreateAssociatedTokenAccount, getAssociatedTokenAddress,createSyncNativeInstruction,getAccount, mintTo, getMint} = require("@solana/spl-token");
const bs58 = require("bs58");
const {struct, u32, u8, uint, BufferLayout, SystemInstructionInputData, getAlloc, ns64} = require("@solana/buffer-layout");
const {publicKey, u64, bool} = require("@solana/buffer-layout-utils");
const {Buffer} = require("buffer");
const axios = require('axios');
const BN = require('bn.js');
const BigNumber = require("bignumber.js");
const { Worker } = require('worker_threads')
const MINIMAL_MARKET_STATE_LAYOUT_V3 = struct([publicKey('eventQueue'), publicKey('bids'), publicKey('asks')]);
const {
    BigNumberish,
    Liquidity,
    LIQUIDITY_STATE_LAYOUT_V4,
    LiquidityPoolKeys,
    LiquidityStateV4,
    MARKET_STATE_LAYOUT_V3,
    MarketStateV3,
    Token,
    TokenAmount,
    SPL_ACCOUNT_LAYOUT,
    MAINNET_PROGRAM_ID,
    TokenAccount,
    Market,
    SPL_MINT_LAYOUT,

} = require('@raydium-io/raydium-sdk');
// const {
//     Keypair,
//     Connection,
//     PublicKey,
//     ComputeBudgetProgram,
//     KeyedAccountInfo,
//     TransactionMessage,
//     VersionedTransaction,
//     LAMPORTS_PER_SOL,
//     SystemProgram,
//     Transaction
// } = require('@solana/web3.js');

// Create a WebSocket connection
const ws = new WebSocket('wss://atlas-mainnet.helius-rpc.com?api-key=d74e211f-952b-43d8-9589-57c8fed34f55');
// const ws = new WebSocket('wss://solana-mainnet.core.chainstack.com/65b4d5f234db72093adf9c9f02853bef');
let buyFlag = false;
const wallet = solanaWeb3.Keypair.fromSecretKey(new Uint8Array(bs58.decode("2ECNwgFxptwS7G99RKCCBxFudgumA7R22zVJdjhD3yeKPDtjMzuo24TxLDm9Hzud4g1dF4cMhyzWmz6RR8GAwF3R")));

const connectionHigh = new Connection('http://va.pixelpilotz.com/', {
    wsEndpoint: 'wss://va.pixelpilotz.com/',
    commitment: 'confirmed',
});

const connection = new Connection('http://va.pixelpilotz.com/', {
    wsEndpoint: 'wss://va.pixelpilotz.com/',
    commitment: 'confirmed',
});

const connectionTransction = new Connection('http://va.pixelpilotz.com/', {
    wsEndpoint: 'wss://va.pixelpilotz.com/',
    commitment: 'confirmed',
});



let newTokeInfoMap = new Map();
const WRAPPED_SOL_MINT = "So11111111111111111111111111111111111111112";
const WRAPPED_SOL_DECIMALS = 9;
const POOL_RAYDIUM = "675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8";
const RAYDIUM_V4 = "5Q544fKrFoe6tsEbD7S8EmxGTJYAKtTVhAW5Q5pge4j1";
const USDT = 'Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB';
const USDC = 'EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v';
const WSOL = 'So11111111111111111111111111111111111111112';
const mainTokenSet = new Set([USDT, USDC, WSOL]);
let solPrice = 140;
const JITO_TIP_ACCOUNT = new Set(['96gYZGLnJYVFmbjzopPSU6QiEV5fGqZNyN9nmNhvrZU5', 'HFqU5x63VTqvQss8hp11i4wVV8bD44PvwucfZ2bU7gRe', 'Cw8CFyM9FkoMi7K7Crf6HNQqf4uEMzpKw6QNghXLvLkY',
    'ADaUMid9yfUytqMBgopwjb2DTLSokTSzL1zt6iGPaS49', 'DfXygSm4jCyNCybVYYK6DwvWqjKee8pbDmJGcLWNDXjh', 'ADuUkR4vqLUMWXxW9gh6D6L8pMSawimctcNZ5pGwDcEt', 'DttWaMuVvTiduZRnguLF7jNxTgiMBZ1hyAumKUiL2KRL',
    '3AVi9Tg9Uo68tJfuvoKvqKNWKkC5wPdSSdeBnizKZ6jT'
]);

const computeBudgetProgramId = 'ComputeBudget111111111111111111111111111111';
let pkList = ["2ECNwgFxptwS7G99RKCCBxFudgumA7R22zVJdjhD3yeKPDtjMzuo24TxLDm9Hzud4g1dF4cMhyzWmz6RR8GAwF3R",]//私钥

//创建线程
// const worker = new Worker('/home/code/sol/pumpdotfun/listenPumpThread.js');


const SET_COMPUTE_UNIT_PRICE_LAYOUT = struct([
  u8('state'),
  u32('microLamports'),
]);


// let log = console.log;
// console.log = function () {
//     let args = Array.from(arguments);
//     let log_prefix = new Date().toLocaleString('zh-CN', {
//         timeZone: 'Asia/Shanghai',
//     }).replaceAll('/', '-');
//     args.unshift(log_prefix + " ->");
//     log.apply(console, args);
// }


let log = console.log;
console.log = function () {
    let args = Array.from(arguments);
    let log_prefix = new Date().toLocaleString('zh-CN', {
        timeZone: 'Asia/Shanghai',
        hour12: false,
        year: 'numeric',
        month: '2-digit',
        day: '2-digit',
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit',
        fractionalSecondDigits: 3  // 添加毫秒数，精确到3位
    }).replaceAll('/', '-');
    args.unshift(log_prefix + " ->");
    log.apply(console, args);
}

let accountInfo = undefined
let associatedTokenAccount =undefined

let latestBlockHashCache = undefined;
let unitPrice = undefined;
setInterval(async () => {
   latestBlockHashCache = await  connection.getLatestBlockhash('confirmed');
    unitPrice = await getPriorityFee('veryHigh')
   // console.log("获取最新latestBlockHash=",latestBlockHashCache)
}, 1000); // 每3秒检查一次



let pump_signer = '39azUYFWPz3VHgKCf3VChUwbpURdCHRxjWVowf5jUJjg'

// getRaydiumSwapInfo()

function sendRequest(ws) {
    const request = {
        jsonrpc: "2.0",
        id: 420,
        method: "transactionSubscribe",
        params: [
            {   failed: false,
                accountInclude:    ["srmqPvymJeFKQ4zGQed1GFppgkRHL9kaELCbyksJtPX","675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8"]
            },
            {
                commitment: "processed",
                encoding: "jsonParsed",
                transactionDetails: "full",
                maxSupportedTransactionVersion: 0
            }
        ]
    };
    ws.send(JSON.stringify(request));
}
ws.on('open', function open() {
    sendRequest(ws);
});

ws.on('message', function incoming(data) {
    const messageStr = data.toString('utf8');
    try {
        const messageObj = JSON.parse(messageStr);
        if(messageObj != undefined && messageObj.params != undefined){
            const result = messageObj.params.result;
            if(result != undefined){
                const logs = result.transaction.meta.logMessages;
                const signature = result.signature; // Extract the signature
                // const accountKeys = result.transaction.transaction.message.accountKeys.map(ak => ak.pubkey);
                // console.log("ws监听到交易signature=",signature,"accountKeys=",accountKeys)

                let tx = result.transaction
                // console.log("tx=",tx)
                if (tx && tx.transaction.message.accountKeys.length === 13 && tx.transaction.message.instructions.length === 6) {
                    // console.log("监听到交易tx=",signature)
                    let transactionsMeta = tx.transaction.message.instructions
                    let signer = tx.transaction.message.accountKeys[0].pubkey.toString()
                    let accountKeys = []
                    transactionsMeta.forEach(transaction => {
                        if (transaction.accounts) {
                            // console.log("transaction.accounts=",transaction.accounts)
                            const keys = transaction.accounts.map(pubkey => pubkey.toString())
                            accountKeys = accountKeys.concat(keys)
                        }
                    })



                    if (signer === pump_signer) {
                        let tokenAddress = accountKeys[8]
                        let marketId = accountKeys[0]
                        console.log('监听到pump创建市场id', "代币地址", accountKeys[8], "市场id", accountKeys[0],"signature=", signature);
                        // if(newTokeInfoMap.get(tokenAddress) != undefined){
                            processListenMarkey(tokenAddress,marketId)
                        // }
                        
                    } else {
                        console.log('监听到普通创建市场id', "代币地址", accountKeys[7], "市场id", accountKeys[0],"signature=", signature);
                        // let tokenAddress = accountKeys[7]
                        // let marketId = accountKeys[0]
                        // const tokenInfo = {
                        //     "tokenAddress": tokenAddress,
                        //     "market_id": marketId,
                        //     "ctime": new Date(),
                        //     "utime": new Date(),
                        //     "openTrade":false,
                        //     "openTradeTime":undefined,
                        //     "burn_gas_owner":[]
                        // }
                        // newTokeInfoMap.set(tokenAddress,tokenInfo);
                        
                    }
                }

                // getRaydiumSwapInfo(tx)
            }
        }
    } catch (e) {
        console.log("ws监听到交易异常error=",e)
    }
});


// const market_pub_key = "srmqPvymJeFKQ4zGQed1GFppgkRHL9kaELCbyksJtPX";
// let marketAddress = new PublicKey(market_pub_key)
// //订阅市场id创建日志
// connectionHigh.onLogs(
//     marketAddress,
//     ({ logs, err, signature }) => {
//         if (err) return;
//             if (logs) {
//                 // console.log("logs=",logs)
//                 if (logs.some(log => log.includes(market_pub_key))) {
//                     checkCreateOpenBookId(signature);
//                 }
//             }
//         },
//     "processed"
// );

// const checkCreateOpenBookId = async (hash) => {
//     try {
//         const tx = await connectionHigh.getParsedTransaction(hash, {
//             'commitment': 'confirmed', 'maxSupportedTransactionVersion': 0
//         });
//         if (tx == null) {
//             await sleep(100);
//             // console.log("查询交易信息为空，循环遍历,hash=",hash)
//             checkCreateOpenBookId(hash);
//             return;
//         }
//         if (tx && tx.transaction.message.accountKeys.length === 13 && tx.transaction.message.instructions.length === 6) {
//             let transactionsMeta = tx.transaction.message.instructions
//             let signer = tx.transaction.message.accountKeys[0].pubkey.toString()
//             let accountKeys = []
//             transactionsMeta.forEach(transaction => {
//                 if (transaction.accounts) {
//                     const keys = transaction.accounts.map(pubkey => pubkey.toBase58())
//                     accountKeys = accountKeys.concat(keys)
//                 }
//             })

//             if (signer === pump_signer) {
//                 console.log(hash, 'pump创建市场id', "代币地址", accountKeys[8], "市场id", accountKeys[0]);
//             } else {
//                 console.log(hash, '普通创建市场id', "代币地址", accountKeys[7], "市场id", accountKeys[0]);
//             }
//         }
//     } catch (error) {
//         console.log("error checkCreateOpenBookId:", hash, error);
//     }
// }

worker.on('message', msg => {
    if ("saveNewToken" == msg.action) {
        //新代币传入
        // console.log("3. 接收到请求=",msg.action)
        let tokenAddress = msg.data.tokenAddress;
        console.log("3. 设置新币newTokeInfoMap，tokenAddress=",tokenAddress)
        newTokeInfoMap.set(tokenAddress, msg.data);
        let pair = msg.data.pair_address
        // processPairBuy(pair,tokenAddress)


        // if(existingOpenBookTokenMap.get(tokenAddress) != undefined && pendingSellTokenMap.get(tokenAddress) == undefined){
        //     console.log("监听到新建token且市场id不为空进行买入tokenAddress=",tokenAddress)
        //     let marketId = existingOpenBookTokenMap.get(tokenAddress)
        //     demo_buy(marketId,tokenAddress)
        // }

        
        
    }
});

// // Function to send a request to the WebSocket server
// function sendRequest(ws) {
//     const request = {
//         jsonrpc: "2.0",
//         id: 420,
//         method: "transactionSubscribe",
//         params: [
//             {   
//                 // failed: false,
//                 accountInclude:    []
//             },
//             {
//                 commitment: "processed",
//                 encoding: "jsonParsed",
//                 transactionDetails: "full",
//                 maxSupportedTransactionVersion: 0
//             }
//         ]
//     };
//     ws.send(JSON.stringify(request));
// }


// ws.on('open', function open() {
//     sendRequest(ws);
// });

// ws.on('message', function incoming(data) {
//     const messageStr = data.toString('utf8');
//     try {
//         const messageObj = JSON.parse(messageStr);
//         if(messageObj != undefined && messageObj.params != undefined){
//             const result = messageObj.params.result;
//             if(result != undefined){
//                 const logs = result.transaction.meta.logMessages;
//                 const signature = result.signature;
//                 // console.log("监听到交易tx=",signature)
//                 let tx = result.transaction
//                 getRaydiumSwapInfo(tx)
//             }
//         }
//     } catch (e) {
//         console.log("ws监听到交易异常error=",e)
//     }
// });

const startWSSpink = () => {
  try {
    //请注意killall -9 node 是强制关闭，不能被监听到，需要使用 killall -15 node 
    process.on('SIGTERM', onExit);
    process.on('SIGINT', onExit);
    function onExit() {
      console.log("跟单系统正在关闭......请稍等,执行相关保存任务中");
      saveNewTokeInfo()
      console.log("执行完毕,正在退出");
      process.exit(0);
    };

  } catch (error) {
    console.log("执行startWSSpink异常，error=", error);
  }


};

// Function to send a request to the WebSocket server




async function processListenMarkey(tokenAddress,marketId) {
    console.log("出现可以冲开盘的token=",tokenAddress)
    // let pairAddress = newTokeInfoMap.get(tokenAddress).pair_address
    // console.log("查到对应的pairAddress=",pairAddress,"tokenAddress=",tokenAddress)
    await demo_buy(marketId,tokenAddress)
    await sleep(60000);
    let pair_address = newTokeInfoMap.get(tokenAddress).pair_address
    console.log("卖出token，查询到pair_address=",pair_address,"tokenAddress=",tokenAddress)
    if(pair_address != undefined){
        await demo_sell(pair_address,tokenAddress)
    }else{
        let pair = Liquidity.getAssociatedId({
            programId: MAINNET_PROGRAM_ID.AmmV4,
            marketId: marketId,
        })
        console.log("卖出token，从newTokeInfoMap获取不到pair_address，查询得到pair_address=",pair.toString(),"tokenAddress=",tokenAddress)
        await demo_sell(pair.toString(),tokenAddress)
    }
}


//等于
function accEqual(arg1, arg2) {
    return new BigNumber(arg1).eq(new BigNumber(arg2));
}

let raydium_buy = async (info, mint, poolKeys) => {
    try {
        const { innerTransaction } = Liquidity.makeSwapFixedInInstruction(
            {
                poolKeys: poolKeys,
                userKeys: {
                    tokenAccountIn: info.associatedTokenAccount,
                    tokenAccountOut: info.tokenAccountAddress,
                    owner: new PublicKey(info.address),
                },
                amountIn: info.buyTokenAmount,
                minAmountOut: 0,
            },
            poolKeys.version
        );

        console.log("innerTransaction=",innerTransaction)


        if(unitPrice === null){
            unitPrice = 2000000
        }

        let instructions = [
            ComputeBudgetProgram.setComputeUnitPrice({
                microLamports: unitPrice * 2,
            }),
            ComputeBudgetProgram.setComputeUnitLimit({ units: 1000000 }),
            createAssociatedTokenAccountIdempotentInstruction(
                new PublicKey(info.address),
                info.tokenAccountAddress,
                new PublicKey(info.address),
                new PublicKey(mint)
            ),
            SystemProgram.transfer({
                //存款wsol
                fromPubkey: new PublicKey(info.address),
                toPubkey: info.associatedTokenAccount,
                lamports: info.buyTokenAmount,
            }),
            createSyncNativeInstruction(info.associatedTokenAccount),
            ...innerTransaction.instructions,
            createCloseAccountInstruction(
                info.associatedTokenAccount,
                new PublicKey(info.address),
                new PublicKey(info.address)
            ),
        ];
        console.log("instructions=",instructions)
        //只有wsol账户不存在时才创建
        if (!info.accountInfo) {
            console.log("wsol账户不存在创建accountInfo")
            instructions.unshift(
                createAssociatedTokenAccountInstruction(
                    new PublicKey(info.address),
                    info.associatedTokenAccount,
                    new PublicKey(info.address),
                    new PublicKey(WSOL)
                )
            );
        }
        console.log("结束创建accountInfo")
        // const latestBlockhash = await connection.getLatestBlockhash({
        //     commitment: "confirmed",
        // });

        const latestBlockhash =latestBlockHashCache;

        console.log("latestBlockhash=",latestBlockhash)

        const messageV0 = new TransactionMessage({
            payerKey: new PublicKey(info.address),
            recentBlockhash: latestBlockhash.blockhash,
            instructions,
        }).compileToV0Message();
        const transaction = new VersionedTransaction(messageV0);
        transaction.sign([info.keypair, ...innerTransaction.signers]);
        console.log("提交交易，transaction=",transaction)
        const hash = await connectionTransction.sendRawTransaction(transaction.serialize());
        console.log("hash", hash);

        checkHash(info, hash);
    } catch (error) {
        if (error.message.indexOf("Transfer: insufficient lamports") != -1) {
            console.log("余额不足");
        }
        console.log("error=",error)
        const nowTokenBalance = await getTokenBalance(info.address, mint);
        // if (!accEqual(info.tokenBalance, nowTokenBalance)) {
        //     info.isSucceed = true;
        //     console.log("购买成功");
        // }
        if(nowTokenBalance>0){
            info.isSucceed = true;
            console.log("购买成功");
        }
    }
}


const checkHash = async (info, hash) => {
    for (let i = 0; i < 2; i++) {
        await sleep(10000)
        if (!info.isSucceed) {
            try {
                await connectionTransction.confirmTransaction(
                    hash,
                    "confirmed"
                );
                info.isSucceed = true
                console.log(`${info.address} 购买成功, hash: ${hash}`);
                // let tokenAddress = info.address
                // let tokenInfo = { "hash": hash, "buyTokenAmount": info.buyTokenAmount}
                // pendingSellTokenMap.set(tokenAddress,tokenInfo)
            } catch (error) {
                console.log("获取交易状态失败", error);
                // checkHash(info, hash);
                return
            }
        }
    }
}
let demo_buy = async (marketId,mint) => {
    let buyTokenAmount = 0.05 * 1e9 //支付数量 //0.0129 0.00584  手续费 0.00006
    // let marketId = 'Gg4XtBcyeoNSQLLYtTuv3hVuZ8CJx1AgXYXchtTm9VSr'; //市场id
    // let mint = '25hAyBQfoDhfWx9ay6rarbgvWGwDdNqcHsXS3jQ3mTDJ' //代币地址

    // 获取所有的key
    // let tokenArray = Object.keys(newTokeInfoMap);
    // console.log("当前token=",mint,"tokenArray=",tokenArray)
    // if(tokenArray.includes(mint)){
        let walletInfo = []
        console.log("当前买入token=",mint)
        for (let i = 0; i < pkList.length; i++) {
            const pk = pkList[i];
            let keypair = Keypair.fromSecretKey(bs58.decode(pk));
            // const balance = await connection.getBalance(keypair.publicKey)

            // console.log(keypair.publicKey.toString(), `${balance / LAMPORTS_PER_SOL} SOL`);

            
            let tokenAccountAddress = await getAssociatedTokenAddress(
                new PublicKey(mint),
                new PublicKey(keypair.publicKey)
            );

            // const tokenBalance = await getTokenBalance(keypair.publicKey, mint);
            // console.log("tokenBalance=",tokenBalance,"mint=",mint,"marketId=",marketId)
            console.log("tokenAccountAddress", tokenAccountAddress,"accountInfo",accountInfo,"associatedTokenAccount=",associatedTokenAccount);

            walletInfo.push(
                {
                    address: keypair.publicKey,
                    keypair,
                    associatedTokenAccount,
                    accountInfo,
                    tokenAccountAddress,
                    buyTokenAmount,
                    isSucceed: false,
                    tokenBalance: 0
                }
            )
        }
        // console.log(walletInfo);
        const market = await getMarketV3(connection, new PublicKey(marketId), 'confirmed');
        console.log("获取到market信息");
        if(market == undefined){
            console.log("获取到market信息为空，直接返回")
            return
        }
        /* let mint, pool
        if (
            market.baseMint.toString() === WSOL ||
            market.baseMint.toString() === USDC
        ) {
            mint = market.quoteMint.toString();
            pool = market.baseMint.toString();
        } else {
            mint = market.baseMint.toString();
            pool = market.quoteMint.toString();
        }
        console.log(mint, pool); */

        let poolKeys = await derivePoolKeys(connection, market, 'confirmed')
        console.log("获取到pair_address=",poolKeys.id.toString())
        if(newTokeInfoMap.get(mint) == undefined){
            const tokenInfo = {
                  "tokenAddress": mint,
                  "creator": undefined,
                  "created_timestamp": undefined,
                  "market_cap": undefined,
                  "ctime": new Date(),
                  "utime": new Date(),
                  "price": undefined,
                  "alreadyBuy":false,
                  "create_time_diff":undefined,
                  "pair_address":undefined
              }
            newTokeInfoMap.set(mint,tokenInfo);
        }
        newTokeInfoMap.get(mint).pair_address = poolKeys.id.toString()

        // if(poolKeys)
        // console.log("baseMint=", poolKeys.baseMint.toString(),"quoteMint=",poolKeys.quoteMint.toString(),"lpMint=",poolKeys.lpMint.toString());

        



        // 直接买入代码
        // const info = walletInfo[0];
        // if (!info.isSucceed) {
        //     buy(info, mint, poolKeys);
        // }


         // 烧gas买入代码
        
        // console.log("开始买入token=",poolKeys.baseMint.toString());
        const info = walletInfo[0];
        if (!info.isSucceed) {
            raydium_buy(info, mint, poolKeys);
        }
        // //这里开始燃烧交易
        // while (true) {
        //     const info = walletInfo[0];
        //     if (!info.isSucceed) {
        //         raydium_buy(info, mint, poolKeys);
        //     } else {
        //         break;
        //     }
        //     await sleep(100);
        // }
    // }

    
    
}

let demo_sell = async (pair,mint) => {
    // let mint = '25hAyBQfoDhfWx9ay6rarbgvWGwDdNqcHsXS3jQ3mTDJ'
    // let pair = '2aPsSVxFw6dGRqWWUKfwujN6WVoyxuhjJaPzYaJvGDDR'
    let walletInfo = []
    for (let i = 0; i < pkList.length; i++) {
        const pk = pkList[i];
        let keypair = Keypair.fromSecretKey(bs58.decode(pk));
        let associatedTokenAccount = await getAssociatedTokenAddress(
            new PublicKey(WSOL),
            keypair.publicKey
        );
        let accountInfo = await connection.getAccountInfo(associatedTokenAccount);

        let tokenAccountAddress = await getAssociatedTokenAddress(
            new PublicKey(mint),
            keypair.publicKey
        );

        let sellAmount = undefined
        for(let i=0;i<10;i++){
            console.log("第",i+1,"次查询sellAmount")
            let tokenAccounts = await connection.getParsedTokenAccountsByOwner(keypair.publicKey, {
                mint: new PublicKey(mint)
            });
            console.log("tokenAccounts=",tokenAccounts);
            if(tokenAccounts.value !=undefined &&  tokenAccounts.value.length>0){
                for (const account of tokenAccounts.value) {
                    sellAmount = new BN(account.account.data.parsed.info.tokenAmount.amount)
                    console.log(mint, `token余额: ${account.account.data.parsed.info.tokenAmount.uiAmount} `);
                }
                break
            }

            await sleep(10000)
        }

        if(sellAmount == undefined){
            console.log("查询到持有token金额为空，直接返回,tokenAddress=",mint)
            return
        }
        
        

        walletInfo.push(
            {
                address: keypair.publicKey,
                keypair,
                associatedTokenAccount,
                accountInfo,
                tokenAccountAddress,
                sellAmount,
                isSucceed: false,
            }
        )
    }

    const info = await connection.getAccountInfo(new PublicKey(pair));
    // console.log(info);
    //解析池子信息
    const poolState = LIQUIDITY_STATE_LAYOUT_V4.decode(info.data);
    /*  let mint
     if (
         poolState.baseMint.toString() === WSOL ||
         poolState.baseMint.toString() === USDC
     ) {
         mint = poolState.quoteMint.toString();
         pool = poolState.baseMint.toString();
     } else {
         mint = poolState.baseMint.toString();
         pool = poolState.quoteMint.toString();
     } */

    let marketId = poolState.marketId.toString();
    //   console.log(mint, marketId);
    let market = await getMinimalMarketV3(connection, new PublicKey(marketId), 'confirmed');

    let poolKeys = createPoolKeys(
        new PublicKey(pair),
        poolState,
        market
    );

    for (let i = 0; i < walletInfo.length; i++) {
        const info = walletInfo[i];
        if (!info.isSucceed) {

            raydium_sell(info, mint, poolKeys)
        }


    }

}

let raydium_sell = async (info, mint, poolKeys) => {
    // console.log("进入到raydium_sell，打印关键日志：tokenAccountIn=",info.tokenAccountAddress,"tokenAccountOut=",info.associatedTokenAccount,"amountIn=",info.sellAmount,"poolKeys=",poolKeys)
    const { innerTransaction } = Liquidity.makeSwapFixedInInstruction(
        {
            poolKeys: poolKeys,
            userKeys: {
                tokenAccountIn: info.tokenAccountAddress,
                tokenAccountOut: info.associatedTokenAccount,
                owner: new PublicKey(info.address),
            },
            amountIn: info.sellAmount,
            minAmountOut: 0,
        },
        poolKeys.version
    );
    console.log("innerTransaction=",innerTransaction)

    let instructions = [
        ComputeBudgetProgram.setComputeUnitPrice({
            microLamports: 300000,
        }),
        ComputeBudgetProgram.setComputeUnitLimit({ units: 200000 }),
        ...innerTransaction.instructions,

        // createCloseAccountInstruction(
        //     info.associatedTokenAccount,
        //     new PublicKey(info.address),
        //     new PublicKey(info.address)
        // ),
    ];
    if (!info.accountInfo) {
        console.log("")
        instructions.unshift(
            createAssociatedTokenAccountInstruction(
                new PublicKey(info.address),
                info.associatedTokenAccount,
                new PublicKey(info.address),
                new PublicKey(WSOL)
            )
        );
    }
    const latestBlockhash = await connection.getLatestBlockhash({
        commitment: "confirmed",
    });
    console.log("raydium_sell,latestBlockhash=",latestBlockhash);

    const messageV0 = new TransactionMessage({
        payerKey: new PublicKey(info.address),
        recentBlockhash: latestBlockhash.blockhash,
        instructions,
    }).compileToV0Message();

    const transaction = new VersionedTransaction(messageV0);
    transaction.sign([info.keypair, ...innerTransaction.signers]);
    const checkSignatures = transaction.signatures;

    const signatureBase58 = bs58.encode(checkSignatures[0])
    console.log(signatureBase58);
    const hash = await connectionTransction.sendRawTransaction(transaction.serialize());
    try {
        await connectionTransction.confirmTransaction(
            hash,
            "confirmed"
        );
        console.log(`${info.address} 卖出成功, hash: ${hash}`);
    } catch (error) {
        console.log(`${info.address}卖出失败,重新发送交易`);
        sell(info, mint, poolKeys)
    }

}

const getTokenBalance = async (walletAddress, mint) => {
    try {
        let tokenAccounts = await connection.getParsedTokenAccountsByOwner(new PublicKey(walletAddress), {
            mint: new PublicKey(mint)
        });
        if (tokenAccounts.value == []) {
            console.log("没有检测到余额");
            return 0
        }
        //console.log(tokenAccounts);
        let tokenAmount
        let tokenAmountBN
        for (const account of tokenAccounts.value) {
            //console.log(account.account.data.parsed.info);
            tokenAmount = account.account.data.parsed.info.tokenAmount.uiAmount;
            tokenAmountBN = new BN(account.account.data.parsed.info.tokenAmount.amount)
            console.log(mint, `token余额: ${tokenAmount} ${tokenAmountBN}`);

        }
        return tokenAmountBN;
    } catch (error) {
        console.log("获取余额失败");
        return 0;
    }
}


function getRaydiumSwapInfo(txn){

  // 判断raydium相关账号的余额增减情况
  const balanceChangeInfo = getBalanceChangeInfo(txn);
  if (!balanceChangeInfo) {
    return null;
  }

  const senderBalanceChanges = getChangInfoByOwner(balanceChangeInfo.balanceChanges, balanceChangeInfo.sender);
  const raydiumBalanceChanges = getChangInfoByOwner(balanceChangeInfo.balanceChanges, '5Q544fKrFoe6tsEbD7S8EmxGTJYAKtTVhAW5Q5pge4j1');

  // console.log("senderBalanceChanges=",senderBalanceChanges,"raydiumBalanceChanges=",raydiumBalanceChanges)
  let raydiumMainTokenChangeInfos = raydiumBalanceChanges.filter((change) => change.token && isMainToken(change.token));
  let raydiumTokenChangeInfos = raydiumBalanceChanges.filter((change) => change.token && !isMainToken(change.token));
  if (raydiumTokenChangeInfos.length === 0 || raydiumMainTokenChangeInfos.length === 0) {
    let account_signer = txn.transaction.message.accountKeys[0].pubkey
    // console.log('未能识别的raydium交易' + balanceChangeInfo.signature,"owner=",account_signer);
    
    let tokenAddress = undefined
    let liquidity = undefined
    if(txn.meta.preTokenBalances != undefined && txn.meta.preTokenBalances.length>0){
        let preTokenBalanceList = txn.meta.preTokenBalances
        // console.log("preTokenBalanceList=",preTokenBalanceList)
        preTokenBalanceList.forEach(item => {
            // console.log("item=",item)
            let owner = item.owner
            let mint = item.mint
            // console.log(`Owner: ${item.owner}, Mint: ${item.mint}`);
            if(owner == RAYDIUM_V4){
                if(isMainToken(mint)){
                    liquidity = mint
                }else{
                    tokenAddress = mint
                }
            }
        });

        if(tokenAddress != undefined && liquidity != undefined){
            // console.log("识别到确认失败的swap交易，swap token=",tokenAddress,"swap signer=",account_signer)
            if(newTokeInfoMap.has(tokenAddress) && newTokeInfoMap.get(tokenAddress).openTrade == false){
                console.log("识别到大佬=",account_signer,"烧gas买入token=",tokenAddress,"txHash=",balanceChangeInfo.signature)
                let burnGasOwnerList = newTokeInfoMap.get(tokenAddress).burn_gas_owner
                burnGasOwnerList.push(account_signer)
                newTokeInfoMap.get(tokenAddress).burn_gas_owner = burnGasOwnerList
            }
        }
    }
    return null;
  }
  let swapTokens = [];
  for (let raydiumTokenChangeInfo of raydiumTokenChangeInfos) {
    // console.log("raydiumTokenChangeInfo=",raydiumTokenChangeInfo)
    if (!raydiumTokenChangeInfo.token || isMainToken(raydiumTokenChangeInfo.token)) {
        // console.log("!raydiumTokenChangeInfo.token || isMainToken(raydiumTokenChangeInfo.token)")
        continue;
    }

    let token = raydiumTokenChangeInfo.token;
    // 判断sender的账户是否有变化
    let senderBalanceChange = senderBalanceChanges.find((senderChange) => senderChange.token === token);
    // 没变化则证明这个交易对只是个中间变量，用户并不是交易这个token
    if (!senderBalanceChange) {
        // console.log("senderBalanceChange为false")
        continue;
    }
    // 确保2个余额变化一个是正数一个是负数 且余额变化不为0
    if (senderBalanceChange.balanceChange * raydiumTokenChangeInfo.balanceChange >= 0) {
        // console.log("senderBalanceChange.balanceChange * raydiumTokenChangeInfo.balanceChange >= 0")
        continue;
    }
    swapTokens.push({token: token, poolBalanceChange: raydiumTokenChangeInfo.balanceChange});
    // console.log("swapTokens=",swapTokens)
  }
  if (swapTokens.length > 1) {
    // console.log('未能识别的swap' + balanceChangeInfo.signature);
    return null;
  }
  let swapToken = swapTokens[0];
  // console.log("swapToken=",swapToken)
  if(swapToken==undefined){
    return null
  }
  // 找出raydium主流币的变化
  let raydiumMainTokenChanges = raydiumMainTokenChangeInfos.filter((change) => isMainToken(change.token));
  let maxChangeMainToken = null;
  let maxChangeMainAmount = 0;
  let maxChangeMainUsdAmount = 0;
  for (let raydiumMainTokenChange of raydiumMainTokenChanges) {
    if (!raydiumMainTokenChange.token) {
        // console.log("!raydiumMainTokenChange.token")
        continue;
    }
    // 变化必须和 token为相反数
    if (!isOpposition(raydiumMainTokenChange.balanceChange,swapToken.poolBalanceChange)) {
        // console.log("!isOpposition(raydiumMainTokenChange.balanceChange,swapToken.poolBalanceChange")
        continue;
    }
    let usdAmount = raydiumMainTokenChange.balanceChange;
    if (raydiumMainTokenChange.token === WSOL) {
      usdAmount = usdAmount * solPrice;
    }
    if (Math.abs(raydiumMainTokenChange.balanceChange) > maxChangeMainUsdAmount) {
      maxChangeMainToken = raydiumMainTokenChange.token;
      maxChangeMainAmount = raydiumMainTokenChange.balanceChange;
      maxChangeMainUsdAmount = Math.abs(usdAmount);
    }
  }

  if (!maxChangeMainToken) {
    // console.log(`未能识别的swap ${balanceChangeInfo.signature} (未发现主流代币)`)
    return null;
  }

  let type = maxChangeMainAmount > 0 ? 0 : 1;
  let amountOut = type === 0 ? maxChangeMainAmount : swapToken.poolBalanceChange;
  let amountIn = type === 0 ? swapToken.poolBalanceChange : maxChangeMainAmount;
  let unitPrice = getUnitPrice(txn);
  // console.log("最终解析结果token=",swapToken.token,"liquidity=",maxChangeMainToken,"type=",type,"amountOut=",amountOut,"amountIn=",amountIn,"unitPrice=",unitPrice)
  if(newTokeInfoMap.has(swapToken.token) && newTokeInfoMap.get(swapToken.token).openTrade == false){
      console.log("识别到token=",swapToken.token,"已经有买入成功交易=",balanceChangeInfo.signature)
      newTokeInfoMap.get(swapToken.token).openTrade = true
      newTokeInfoMap.get(swapToken.token).openTradeTime = new Date()
  }
  return {
    token: swapToken.token,
    liquidity: maxChangeMainToken,
    type: type,
    amountOut: amountOut,
    amountIn: Math.abs(amountIn),
    unitPrice: unitPrice,
    dexType: 'raydium',
    extraInfo: null,
  }
}


function getBalanceChangeInfo(parsedTxn) {
    // 获取交易的余额变化
    let fee = parsedTxn.meta.fee;
    let postTokenBalances = parsedTxn.meta.postTokenBalances;
    let preTokenBalances = parsedTxn.meta.preTokenBalances;
    let postTokenBalanceMap = tokenBalanceToMap(postTokenBalances);
    let preTokenBalanceMap = tokenBalanceToMap(preTokenBalances);
    // console.log("parsedTxn.transaction.message.accountKeys[0]=",parsedTxn.transaction.message.accountKeys[0])
    let sender = parsedTxn.transaction.message.accountKeys[0].pubkey;
    // let senders = getSigner(parsedTxn);
    let jitoTip = getJitoTip(parsedTxn, sender);
    let balancaChanges = []
    // 获取所有tokenBalance的组合 (即所有的owner+mint的组合)
    let allBalanceKeys = mergeKeys(postTokenBalanceMap, preTokenBalanceMap);
    for (let allBalanceKey of allBalanceKeys) {
        let post = postTokenBalanceMap[allBalanceKey];
        let pre = preTokenBalanceMap[allBalanceKey];
        // 前后两个余额相减，得到余额变化
        let preBalance = pre ? pre.uiTokenAmount.uiAmount : 0;
        let postBalance = post ? post.uiTokenAmount.uiAmount : 0;
        let balanceChange = postBalance - preBalance;
        if (!balanceChange) {
            continue;
        }
        // pre获取post可能为空，这里就取有的那个
        let owner = post ? post.owner : pre.owner;
        let token = post ? post.mint : pre.mint
        balancaChanges.push({
            "token": token,
            "owner": owner,
            "balanceChange": balanceChange
        })
    }

    let preBalances = parsedTxn.meta.preBalances;
    let postBalances = parsedTxn.meta.postBalances;
    let accounts = parsedTxn.transaction.message.accountKeys;
    let createAccountAmount = getCreateAccountAmount(parsedTxn);
    // 分析sol余额变化
    for (let i = 0; i < preBalances.length; i++) {
        let pre = preBalances[i];
        let post = postBalances[i];
        let balanceChange = post - pre;
        // 如果owner为交易发送者，则余额变化加上手续费 (因为这里的余额已经是扣除手续费了，为确保准确，这里加回去)
        let owner = accounts[i].pubkey;
        if (owner === sender) {
            balanceChange += fee;
            balanceChange += jitoTip;
            balanceChange += createAccountAmount;
        }

        if (!balanceChange) {
            continue;
        }


        // let token = WSOL;
        balancaChanges.push({
            "token": null,
            "owner": owner,
            "balanceChange": balanceChange / 10 ** 9
        })
    }


    return {
        "signature": parsedTxn.transaction.signatures[0],
        "sender": sender,
        "blockTime": parsedTxn.blockTime,
        "slot" : parsedTxn.slot,
        "balanceChanges": balancaChanges,
        "fee": fee,
        "jitoTip": jitoTip
    }
}




//通过marketId获取openbook market数据
async function getMinimalMarketV3(
    connection,
    marketId,
    commitment,
) {
    const marketInfo = await connection.getAccountInfo(marketId, {
        commitment,
        dataSlice: {
            offset: MARKET_STATE_LAYOUT_V3.offsetOf('eventQueue'),
            length: 32 * 3,
        },
    });
    return MINIMAL_MARKET_STATE_LAYOUT_V3.decode(marketInfo.data);
}

async function getMarketV3(
    connection,
    marketId,
    commitment,
) {
    let marketResult = undefined
    for(let i =0;i<10;i++){
         console.log("---第",i+1,"次循环")
         const marketInfo = await connection.getAccountInfo(marketId, {
             commitment,
         });
         if (marketInfo != null) {
             const marketDeco = MARKET_STATE_LAYOUT_V3.decode(marketInfo.data);
             if (marketDeco.baseMint) {
                 marketResult =  marketDeco
                 break
             }
         }
         // await sleep(20)
    }

    return marketResult
}

function createPoolKeys(
    id,
    accountData,
    minimalMarketLayoutV3,
) {
    let obj = {
        id,
        baseMint: accountData.baseMint,
        quoteMint: accountData.quoteMint,
        lpMint: accountData.lpMint,
        baseDecimals: accountData.baseDecimal.toNumber(),
        quoteDecimals: accountData.quoteDecimal.toNumber(),
        lpDecimals: 5,
        version: 4,
        programId: MAINNET_PROGRAM_ID.AmmV4,
        authority: Liquidity.getAssociatedAuthority({
            programId: MAINNET_PROGRAM_ID.AmmV4,
        }).publicKey,
        openOrders: accountData.openOrders, //开放订单的地址
        targetOrders: accountData.targetOrders, //目标订单地址
        baseVault: accountData.baseVault,
        quoteVault: accountData.quoteVault,
        marketVersion: 3,
        marketProgramId: accountData.marketProgramId,
        marketId: accountData.marketId,
        marketAuthority: Market.getAssociatedAuthority({
            programId: accountData.marketProgramId,
            marketId: accountData.marketId,
        }).publicKey,
        marketBaseVault: accountData.baseVault,
        marketQuoteVault: accountData.quoteVault,
        marketBids: minimalMarketLayoutV3.bids,
        marketAsks: minimalMarketLayoutV3.asks,
        marketEventQueue: minimalMarketLayoutV3.eventQueue,
        withdrawQueue: new PublicKey('11111111111111111111111111111111'),
        lpVault: new PublicKey('11111111111111111111111111111111'),
        lookupTableAccount: PublicKey.default,
    };
    // console.log(obj);
    return obj
}
async function getMintData(connection, mint, commitment) {
    const accountInfo = await connection.getAccountInfo(mint, commitment);
    const mintData = SPL_MINT_LAYOUT.decode(accountInfo.data);
    return mintData;
}
const SERUM_MARKET = MAINNET_PROGRAM_ID.SERUM_MARKET;
async function derivePoolKeys(
    connection,
    marketLayoutV3,
    commitment,
) {
    const baseMint = marketLayoutV3.baseMint;
    const quoteMint = marketLayoutV3.quoteMint;

    // const baseMintData = await getMintData(connection, baseMint, commitment);
    // const quoteMintData = await getMintData(connection, quoteMint, commitment);

    const [baseMintData, quoteMintData] = await Promise.all([
      getMintData(connection, baseMint, commitment),
      getMintData(connection, quoteMint, commitment)
    ]);


    const quoteDecimals = quoteMintData.decimals;
    const baseDecimals = baseMintData.decimals;

    

    const authority = Liquidity.getAssociatedAuthority({
        programId: MAINNET_PROGRAM_ID.AmmV4,
    }).publicKey;

    const poolKeys = {
        id: Liquidity.getAssociatedId({
            programId: MAINNET_PROGRAM_ID.AmmV4,
            marketId: marketLayoutV3.ownAddress,
        }),
        baseMint: baseMint,
        quoteMint,
        lpMint: Liquidity.getAssociatedLpMint({
            programId: MAINNET_PROGRAM_ID.AmmV4,
            marketId: marketLayoutV3.ownAddress,
        }),
        baseDecimals: baseDecimals,
        quoteDecimals: quoteDecimals,
        lpDecimals: baseDecimals, //这里不一样  代码是 5 还是 baseDecimals
        version: 4,
        programId: MAINNET_PROGRAM_ID.AmmV4,
        authority: authority,
        openOrders: Liquidity.getAssociatedOpenOrders({
            programId: MAINNET_PROGRAM_ID.AmmV4,
            marketId: marketLayoutV3.ownAddress,
        }),
        targetOrders: Liquidity.getAssociatedTargetOrders({
            programId: MAINNET_PROGRAM_ID.AmmV4,
            marketId: marketLayoutV3.ownAddress,
        }),
        baseVault: Liquidity.getAssociatedBaseVault({
            programId: MAINNET_PROGRAM_ID.AmmV4,
            marketId: marketLayoutV3.ownAddress,
        }),
        quoteVault: Liquidity.getAssociatedQuoteVault({
            programId: MAINNET_PROGRAM_ID.AmmV4,
            marketId: marketLayoutV3.ownAddress,
        }),
        marketVersion: 3,
        marketProgramId: MAINNET_PROGRAM_ID.OPENBOOK_MARKET,
        marketId: marketLayoutV3.ownAddress,
        marketAuthority: Market.getAssociatedAuthority({
            programId: SERUM_MARKET,
            marketId: marketLayoutV3.ownAddress,
        }).publicKey,
        marketBaseVault: marketLayoutV3.baseVault,
        marketQuoteVault: marketLayoutV3.quoteVault,
        marketBids: marketLayoutV3.bids,
        marketAsks: marketLayoutV3.asks,
        marketEventQueue: marketLayoutV3.eventQueue,
        withdrawQueue: new PublicKey('11111111111111111111111111111111'),
        lpVault: new PublicKey('11111111111111111111111111111111'),
        lookupTableAccount: PublicKey.default,
    };
    return poolKeys;
}

function getBalanceKey(tokenBalance) {
    return tokenBalance.owner + '_' + tokenBalance.mint;
}


function tokenBalanceToMap(tokenBalances) {
    // 将token余额数组 转换为
    let result = {};
    for (let balance of tokenBalances) {
        let key = getBalanceKey(balance);
        result[key] = balance;
    }
    return result;
}

function instructionForEach(parsedTxn, callback) {
    let instructions = parsedTxn.transaction.message.instructions;
    for (let instruction of instructions) {
        callback(instruction);
    }
    // 再遍历innerInstructions
    let innerInstructions = parsedTxn.meta.innerInstructions;
    for (let innerInstruction of innerInstructions) {
        let instructions = innerInstruction.instructions;
        for (let instruction of instructions) {
            callback(instruction);
        }
    }
}

function getJitoTip(parsedTxn, sender) {
    let fee = 0;
    instructionForEach(parsedTxn, (instruction) => {
        let type = instruction?.parsed?.type;
        if (!type || type !== 'transfer') {
            return;
        }
        let source = instruction.parsed.info.source;
        let destination = instruction.parsed.info.destination;
        if (source !== sender) {
            return;
        }
        if (!JITO_TIP_ACCOUNT.has(destination)) {
            return;
        }
        fee += instruction.parsed.info.lamports;
    });
    return fee;
}


function mergeKeys(...objs) {
    // 使用reduce方法遍历所有对象，累积它们的键
    const allKeys = objs.reduce((acc, obj) => {
        return acc.concat(Object.keys(obj));
    }, []);

    // 使用Set去重，然后转换为数组
    return [...new Set(allKeys)];
}

function getCloseAccountAmountInInstruction(instruction, balanceChanges) {
    let programId = instruction.programId;
    if (programId !== '11111111111111111111111111111111') {
        return 0;
    }
    let type = instruction?.parsed?.type;
    if (type !== 'closeAccount') {
        return 0;
    }
    let changeInfo = getChangInfoByOwner(balanceChanges, instruction.parsed.info.account);
    return changeInfo?.balanceChange;
}

function getCreateAccountAmountInInstruction(instruction, parsedTxn) {
    // console.log("instruction.programId",instruction.programId)
    let programId = instruction.programId;
    if (programId !== '11111111111111111111111111111111') {
        return 0;
    }
    let type = instruction?.parsed?.type;
    if (type !== 'createAccount') {
        return 0;
    }
    let account = instruction.parsed.info.newAccount;
    // 获取这个账号变动后的sol余额
    return getSolBalanceInParseTxn(parsedTxn, account)
    // return instruction.parsed.info.lamports;
}

function getSolBalanceInParseTxn(parsedTxn, account) {
    let preBalances = parsedTxn.meta.preBalances;
    let postBalances = parsedTxn.meta.postBalances;
    let accounts = parsedTxn.transaction.message.accountKeys;
    for (let i = 0; i < preBalances.length; i++) {
        // console.log("accounts[i].pubkey=",accounts[i].pubkey)
        let owner = accounts[i].pubkey;
        if (owner === account) {
            return postBalances[i] - preBalances[i];
        }
    }
    return null;

}

function getCreateAccountAmount(parsedTx) {
    let lamports = 0;
    instructionForEach(parsedTx, (instruction) => {
        lamports += getCreateAccountAmountInInstruction(instruction, parsedTx);
    });
    return lamports;
}

function getChangInfoByOwner(balanceChanges, owner) {
    let result = [];
    for (let change of balanceChanges) {
        if (change.owner === owner) {
            result.push(change);
        }
    }
    return result;
}

function isMainToken(token) {
    return mainTokenSet.has(token);
}

function isOpposition(num1, num2) {
  return num1 * num2 < 0;
}

function getUnitPrice(txn) {
  let unitPrice = null;
  instructionForEach(txn, (instruction) => {
    // if (!(instruction as PartiallyDecodedInstruction)) {
    //   return;
    // }
    if (!instruction) {
      return;
    }

    let programId = instruction.programId;
    if (programId !== computeBudgetProgramId) {
      return;
    }
    let data = instruction.data;
    if (getDimension(data) === 3) {
      unitPrice = SET_COMPUTE_UNIT_PRICE_LAYOUT.decode(bs58.decode(data)).microLamports;
    }
  });
  return unitPrice;
}

function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

function getDimension(data) {
  let uint8Arr = bs58.decode(data);
  return uint8Arr[0];
}
// // Function to send a request to the WebSocket server
// function sendRequest(ws) {
//     const request = {
//         jsonrpc: "2.0",
//         id: 420,
//         method: "transactionSubscribe",
//         params: [
//             {   failed: false,
//                 accountInclude:    ["6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P"]
//             },
//             {
//                 commitment: "processed",
//                 encoding: "jsonParsed",
//                 transactionDetails: "full",
//                 maxSupportedTransactionVersion: 0
//             }
//         ]
//     };
//     ws.send(JSON.stringify(request));
// }

// ws.on('open', function open() {
//     sendRequest(ws);
// });

// ws.on('message', function incoming(data) {
//     const messageStr = data.toString('utf8');
//     try {
//         const messageObj = JSON.parse(messageStr);
        
//         const result = messageObj.params.result;

//         const logs = result.transaction.meta.logMessages;
//         const signature = result.signature; // Extract the signature
//         const accountKeys = result.transaction.transaction.message.accountKeys.map(ak => ak.pubkey);

//         if (logs && logs.some(log => log.includes('Program log: Instruction: InitializeMint2'))) {
//             // console.log("accountKeys=",result.transaction.transaction.message.accountKeys)
//             console.log('---------------------New pump.fun token!');
//             // console.log('tx:', signature);
//             // console.log('Creator:', accountKeys[0]);
//             console.log('Token:', accountKeys[1]);
//             // console.log('Bonding Curve:', accountKeys[3]);
//             // console.log(' Associated Bonding Curve:', accountKeys[4]);
//             let tokenAddress = accountKeys[1]
//             let bonding_curve = accountKeys[3]
//             let ata_bonding_curve = accountKeys[4]
//             processTx(tokenAddress,bonding_curve,ata_bonding_curve)
            
           

//             // Log the first and second account keys if they exist

//         }
//     } catch (e) {

//     }
// });

// async function processTx(tokenAddress,bonding_curve,ata_bonding_curve) {
//     await buy(tokenAddress,bonding_curve,ata_bonding_curve)
//     await sleep(3000)
//     // await sell(tokenAddress,bonding_curve,ata_bonding_curve)
// }



async function buy(tokenAddress, bondingCurve, ataBondingCurve) {
    console.log(wallet.publicKey.toBase58())
    // await decodeBondingCurve(bondingCurve)
    // const amount = BigInt(3058049999999) // 购买数量 必须必须要对得上，要不会报错BigInt、BN
    // const maxSolCost = BigInt(95301789) // 最多sol成本 = 购买付出的sol + pump费用1% + 创建ATA费用 + 小费
    
    // 发送请求获取必要的数据
    const [latestBlockhash, unitPrice, boundingCurveAccount] = await Promise.all([
      // 获取最新的区块hash
      connection.getLatestBlockhash('confirmed'),
      // 获取优先费
      getPriorityFee('veryHigh'),
      // 获取boundingCurve账户
      connection.getAccountInfo(new PublicKey(bondingCurve), 'processed')
    ]);

    if (!boundingCurveAccount) {
      console.log("Bounding curve account not found, token: ",tokenAddress,"boundingCurveAccount=",boundingCurveAccount);
      return null;
    }
    const buySolAmount = 0.001; // // 总共最多出多少sol;
    // 计算滑点
    const boundingCurveData = PUMP_BONDING_CURVE_LAYOUT.decode(boundingCurveAccount.data);
    const solInLamports = buySolAmount * LAMPORTS_PER_SOL;
    const maxSolCost = Math.floor(solInLamports * (1 + (100 / 100)));
    const tokenOut = Math.floor(Number(
      accMul(solInLamports,
        accDiv(boundingCurveData.virtualTokenReserves.toNumber(),
        boundingCurveData.virtualSolReserves.toNumber())
      )
    ));

    // const amount = BigInt(350547484170) // 购买的准确数量 = 预估得到数量 = 购买金额sol * virtualSolReserves / realTokenReserves
    // const maxSolCost = BigInt(0.001 * LAMPORTS_PER_SOL); // // 总共最多出多少sol

    // 为要传递到传输指令的数据创建缓冲区
    let instructionData = Buffer.alloc(8 + 8); // uint64 + uint64
    instructionData.writeBigUInt64LE(tokenOut, 0); // 将指令索引写入缓冲区
    instructionData.writeBigUInt64LE(maxSolCost, 8); // 将转账金额写入缓冲区
    // console.log(instructionData.toString('hex'))
    instructionData = new Buffer.from('66063d1201daebea' + instructionData.toString('hex'), 'hex') // 前面16位是固定的？
    // console.log(instructionData.toString('hex'))

    // instructionData = Buffer.from('66063d1201daebea7fe4fb01c80200009d30ae0500000000', 'hex')

    // 获取ATA公钥地址
    const mint = new PublicKey(tokenAddress)
    const associatedToken = splToken.getAssociatedTokenAddressSync(mint, wallet.publicKey);
    console.log('ATA:', associatedToken.toBase58())

    // pump买入指令
    const ixBuy = new TransactionInstruction({
        keys: [
            {pubkey: new PublicKey('4wTV1YmiEkRvAtNtsSGPtUrqRYQMe5SKy2uB4Jjaxnjf'), isSigner: false, isWritable: false}, // #1 Global
            {pubkey: new PublicKey('68yFSZxzLWJXkxxRGydZ63C6mHx1NLEDWmwN9Lb5yySg'), isSigner: false, isWritable: true}, // #2 Fee Recipient
            {pubkey: mint, isSigner: false, isWritable: false}, // #3 - Mint (代币合约地址) // 3sbhJE8fW89xCQ3rWCZLe8R3qKRRg8MQUxtu2rpQbz5V
            {pubkey: new PublicKey(bondingCurve), isSigner: false, isWritable: true}, // #4 - Bonding Curve  // DeSLUiWSNUaDUPdsLpuAXvKv6ArRZtCTGB95PNqVptb6
            {pubkey: new PublicKey(ataBondingCurve), isSigner: false, isWritable: true}, // #5 - Associated Bonding Curve  // 4uv6dK16oPbyDXmzXbHXE73yLWD7xNGsxeJXS6PgpHri
            {pubkey: associatedToken, isSigner: false, isWritable: true}, // #6 - Associated User （ATA地址） // 92Zx6fobNnZgt1YvsmQDSTYc9wn9rH15s5BRFdkYjtHv
            {pubkey: wallet.publicKey, isSigner: false, isWritable: true}, // #7 - User // 8zQtpDQLoUYNwLyx8cmdAh5sPeSnUf8Dgc6YT712E3n5
            {pubkey: new PublicKey('11111111111111111111111111111111'), isSigner: false, isWritable: false}, // #8 - System Program
            {pubkey: new PublicKey('TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA'), isSigner: false, isWritable: false}, // #9 - Token Program
            {pubkey: new PublicKey('SysvarRent111111111111111111111111111111111'), isSigner: false, isWritable: false}, // #10 - Rent
            {pubkey: new PublicKey('Ce6TQqeHC9p8KetsN6JsjHK7UTZk7nasjjnr7XxXp9F1'), isSigner: false, isWritable: false}, // #11 - Event Authority
            {pubkey: new PublicKey('6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P'), isSigner: false, isWritable: false}, // #12 - pump.fun Program
            // {pubkey: wallet.publicKey, isSigner: false, isWritable: true}, // receiver
        ],
        programId: new PublicKey('6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P'),
        data: instructionData,
    });

    const PRIORITY_RATE = 100; // MICRO_LAMPORTS 于设置microLamports中的优先费率；在此示例中，为 100 个 microLamport（1 个 microLamport = 0.000001 个lamport）。
    // 获取优先费
    // let unitPrice = await getPriorityFee('high');
    console.log("unitPrice=",unitPrice)
    const ixBudgetPrice = solanaWeb3.ComputeBudgetProgram.setComputeUnitPrice({microLamports: unitPrice})
    const ixBudgetLimit = solanaWeb3.ComputeBudgetProgram.setComputeUnitLimit({units: 1000000})

    // 创建ATA，重复创建会失败
    const ixATA = splToken.createAssociatedTokenAccountInstruction(
        wallet.publicKey,
        associatedToken,
        wallet.publicKey,
        mint,
    )

    // 创建ATA，重复创建会成功
    const ixATAIdempotent = splToken.createAssociatedTokenAccountIdempotentInstruction(
        wallet.publicKey,
        associatedToken,
        wallet.publicKey,
        mint,
    )

    // 所有交易指令
    let allInstructions = [
        ixBudgetPrice,
        ixBudgetLimit,
        ixATAIdempotent,
        ixBuy,
    ]

    // const latestBlockhash = await connection.getLatestBlockhash({commitment: "confirmed"});
    const transaction = new VersionedTransaction(
        new TransactionMessage({
            payerKey: wallet.publicKey,
            // recentBlockhash: PublicKey.default.toString(),
            recentBlockhash: latestBlockhash.blockhash,
            instructions: allInstructions,
        }).compileToV0Message()
    );

    transaction.sign([wallet])

    // // 模拟交易
    // const simulation = await connection.simulateTransaction(transaction, {
    //     replaceRecentBlockhash: true,
    //     sigVerify: false,
    // });
    // console.log("simulation=",transaction)

    const signature = await connection.sendRawTransaction(transaction.serialize(), {skipPreflight: true,});
    console.log('tx:', signature)
    console.log(`https://explorer.solana.com/tx/${signature}?cluster=devnet`)
}

/**
 * 卖出
 * @param {string} tokenAddress     代币合约地址
 * @param {string} bondingCurve     Bonding Curve合约地址
 * @param {string} ataBondingCurve  Bonding Curve 的ATA地址
 * */
async function sell(tokenAddress, bondingCurve, ataBondingCurve) {
    console.log(wallet.publicKey.toBase58())
    const amount = BigInt(58049999999) // 卖多少数量 类型必须必须要对得上，要不会报错BigInt、BN
    const minSolOutput = BigInt(0.0001 * LAMPORTS_PER_SOL) // 最多得到多少sol

    // 第1种
    // 为要传递到传输指令的数据创建缓冲区
    let instructionData = Buffer.alloc(8 + 8); // uint64 + uint64
    instructionData.writeBigUInt64LE(amount, 0); // 将指令索引写入缓冲区
    instructionData.writeBigUInt64LE(minSolOutput, 8); // 将转账金额写入缓冲区
    // console.log(instructionData.toString('hex'))
    //  33e685a4017f83ad 25577a400b170000c5deb42b00000000 // 25337093838629、733273797
    instructionData = new Buffer.from('33e685a4017f83ad' + instructionData.toString('hex'), 'hex') // 前面16位是固定的？
    console.log(instructionData.toString('hex'))

    // instructionData = Buffer.from('33e685a4017f83ad25577a400b170000c5deb42b00000000', 'hex')

    // 获取ATA公钥地址
    const mint = new PublicKey(tokenAddress)
    const associatedToken = splToken.getAssociatedTokenAddressSync(mint, wallet.publicKey);
    console.log('ATA:', associatedToken.toBase58())

    // 获取优先费
    let unitPrice = await getPriorityFee('veryHigh')
    console.log("unitPrice=",unitPrice)
    // pump买入指令
    const ixBuy = new TransactionInstruction({
        keys: [
            {pubkey: new PublicKey('4wTV1YmiEkRvAtNtsSGPtUrqRYQMe5SKy2uB4Jjaxnjf'), isSigner: false, isWritable: false}, // #1 Global
            {pubkey: new PublicKey('68yFSZxzLWJXkxxRGydZ63C6mHx1NLEDWmwN9Lb5yySg'), isSigner: false, isWritable: true}, // #2 Fee Recipient
            {pubkey: mint, isSigner: false, isWritable: false}, // #3 - Mint (代币合约地址) 3sbhJE8fW89xCQ3rWCZLe8R3qKRRg8MQUxtu2rpQbz5V
            {pubkey: new PublicKey(bondingCurve), isSigner: false, isWritable: true}, // #4 - Bonding Curve  // DeSLUiWSNUaDUPdsLpuAXvKv6ArRZtCTGB95PNqVptb6
            {pubkey: new PublicKey(ataBondingCurve), isSigner: false, isWritable: true}, // #5 - Associated Bonding Curve  // 4uv6dK16oPbyDXmzXbHXE73yLWD7xNGsxeJXS6PgpHri
            {pubkey: associatedToken, isSigner: false, isWritable: true}, // #6 - Associated User （ATA地址） // 92Zx6fobNnZgt1YvsmQDSTYc9wn9rH15s5BRFdkYjtHv
            {pubkey: wallet.publicKey, isSigner: false, isWritable: true}, // #7 - User // 8zQtpDQLoUYNwLyx8cmdAh5sPeSnUf8Dgc6YT712E3n5
            {pubkey: new PublicKey('11111111111111111111111111111111'), isSigner: false, isWritable: false}, // #8 - System Program
            {pubkey: new PublicKey('ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL'), isSigner: false, isWritable: false}, // #9 - Associated Token Program
            {pubkey: new PublicKey('TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA'), isSigner: false, isWritable: false}, // #10 - Token Program
            {pubkey: new PublicKey('Ce6TQqeHC9p8KetsN6JsjHK7UTZk7nasjjnr7XxXp9F1'), isSigner: false, isWritable: false}, // #11 - Event Authority
            {pubkey: new PublicKey('6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P'), isSigner: false, isWritable: false}, // #12 - pump.fun Program
            // {pubkey: wallet.publicKey, isSigner: false, isWritable: true}, // receiver
        ],
        programId: new PublicKey('6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P'),
        data: instructionData,
    });

    // 小费
    const PRIORITY_RATE = 100; // MICRO_LAMPORTS 于设置microLamports中的优先费率；在此示例中，为 100 个 microLamport（1 个 microLamport = 0.000001 个lamport）。
    const ixBudgetPrice = solanaWeb3.ComputeBudgetProgram.setComputeUnitPrice({microLamports: unitPrice})
    const ixBudgetLimit = solanaWeb3.ComputeBudgetProgram.setComputeUnitLimit({units: 1000000})

    // 全部卖出后关闭ATA
    const ixCloseATA = splToken.createCloseAccountInstruction(associatedToken, wallet.publicKey, wallet.publicKey)

    // 所有交易指令
    let allInstructions = [
        // ixBudgetPrice,
        // ixBudgetLimit,
        ixBuy,
        ixCloseATA, // close ata放在最后面
    ]

    // 构建交易
    const latestBlockhash = await connection.getLatestBlockhash({commitment: "confirmed"});
    const transaction = new VersionedTransaction(
        new TransactionMessage({
            payerKey: wallet.publicKey,
            // recentBlockhash: PublicKey.default.toString(),
            recentBlockhash: latestBlockhash.blockhash,
            instructions: allInstructions,
        }).compileToV0Message()
    );

    transaction.sign([wallet])

    // 模拟交易
    const simulation = await connection.simulateTransaction(transaction, {
        replaceRecentBlockhash: true,
        sigVerify: false,
    });
    console.log(simulation)

    const signature = await connection.sendRawTransaction(transaction.serialize(), {skipPreflight: true,});
    console.log('tx:', signature)
    console.log(`https://explorer.solana.com/tx/${signature}?cluster=devnet`)
}

function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
};

async function getPriorityFeeAdaptive(priority) {
    let unitPrice = await this.getPriorityFee(priority);
    if (!unitPrice) {
        unitPrice = this.config.unitPrice;
    } else if (unitPrice > this.config.unitPriceMax) {
        console.warn(`优先费高于上限, 设置为最大值: ${this.config.unitPriceMax}`);
        unitPrice = this.config.unitPriceMax;
    }
    return unitPrice;
}

async function getPriorityFee(level) {
    const heliusURL = `https://mainnet.helius-rpc.com/?api-key=76af5ac8-7886-41ef-9a46-0d7576fefd4a`;
    const data = {
        'jsonrpc': '2.0',
        'id': '1',
        'method': 'getPriorityFeeEstimate',
        'params': [{
            'accountKeys': ['JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4'],
            'options': {
              'includeAllPriorityFeeLevels': true,
            },
        }],
    };

    try {
        const response = await axios.post(heliusURL, data);
        const priorityFeeLevels = response.data?.result?.priorityFeeLevels;
        // console.log("priorityFeeLevels=",priorityFeeLevels)
        let result = priorityFeeLevels ? priorityFeeLevels[level] : null;
        if (!result) {
            console.error(`获取优先费用失败: ${JSON.stringify(response.data)}`);
            return null;
        }
        // console.info(`获取优先费用成功: ${result}, high: ${priorityFeeLevels.high}, veryHigh: ${priorityFeeLevels.veryHigh}`);
        return Number(Number(result).toFixed(0));
    } catch (error) {
        console.error('获取优先费出现异常', error);
        return null;
    }
}


//初始化文本中的代卖出代币信息
const initNewTokeInfo = async () => {
  //同步的方式读取文件
  const data = fs.readFileSync('newTokeInfo.json', { encoding: 'utf8', flag: 'r' });
  //还原
  newTokeInfoMap = new Map(JSON.parse(data));
  console.log("初始化新token信息成功, 市场数量=", newTokeInfoMap.size);

}


//初始化文本中的代卖出代币信息
const initAccountInfo = async () => {
    let keypair = Keypair.fromSecretKey(bs58.decode(pkList[0]));
    // console.log("keypair=",keypair)
    associatedTokenAccount = await getAssociatedTokenAddress(
        new PublicKey(WSOL),
        keypair.publicKey
    );
    console.log("associatedTokenAccount=",associatedTokenAccount)
    accountInfo = await connection.getAccountInfo(associatedTokenAccount);
    console.log("accountInfo=",accountInfo)

}

//保存待卖出代币信息到本地文本。
function saveNewTokeInfo() {
  try {
    fs.writeFileSync('newTokeInfo.json', JSON.stringify([...newTokeInfoMap]));
    console.log("保存新token到本地已完成");
  } catch (e) {
    logger.error("保存新token到本地失败", e);
  }
};


async function main() {

    // await initNewTokeInfo()
    await initAccountInfo()
    startWSSpink()
    // pair 2sxbtkkkafHpdsjxSdc7VyaxRPmCNcEJTp8TbspPSdN4
    // demo_buy("AASWfPms4w3CVLxhAbc2fR9GXoSw2uSc2MY9GL4FGtLq","B7ziNCMFgKjdmaUT34Vjph1F8CSfmo6FDXcU9jN9Lu2Q")
    // demo_sell("VFwwzi1gJXsBu4AY4ErNuBa44V6FRjHvi8r914nodwu","Et9mZQHZBJxt6jUGwLBUWFufLu5ifrVBdmAx3r4Tpump")


}


main();

// const market_pub_key = "srmqPvymJeFKQ4zGQed1GFppgkRHL9kaELCbyksJtPX";
// let marketAddress = new PublicKey(market_pub_key)
// //订阅市场id创建日志
// connection.onLogs(
//     marketAddress,
//     ({ logs, err, signature }) => {
//         if (err) return;
//         if (logs) {
//             if (logs.some(log => log.includes(market_pub_key))) {
//                 checkCreateMarketId(signature);
//             }
//         }
//     },
//     "confirmed"
// );




// let pump_signer = '39azUYFWPz3VHgKCf3VChUwbpURdCHRxjWVowf5jUJjg'
// const checkCreateOpenBookId = async (hash) => {
//     try {

//         const tx = await connection.getParsedTransaction(hash, {
//             'commitment': 'confirmed', 'maxSupportedTransactionVersion': 0
//         });
//         if (tx == null) {
//             await sleep(100);
//             checkCreateOpenBookId(hash);
//             return;
//         }
//         if (tx && tx.transaction.message.accountKeys.length === 13 && tx.transaction.message.instructions.length === 6) {
//             console.log("监听到交易tx=",hash)
//             let transactionsMeta = tx.transaction.message.instructions
//             let signer = tx.transaction.message.accountKeys[0].pubkey.toString()
//             let accountKeys = []
//             transactionsMeta.forEach(transaction => {
//                 if (transaction.accounts) {
//                     const keys = transaction.accounts.map(pubkey => pubkey.toBase58())
//                     accountKeys = accountKeys.concat(keys)
//                 }
//             })

//             if (signer === pump_signer) {
//                 console.log(hash, 'pump创建市场id', "代币地址", accountKeys[8], "市场id", accountKeys[0],"signature=",hash);
//             } else {
//                 console.log(hash, '普通创建市场id', "代币地址", accountKeys[7], "市场id", accountKeys[0],"signature=",hash);
//             }
//         }
//     } catch (error) {
//         console.log("error checkCreateOpenBookId:", hash, error);
//     }
// }


// sendRequest(ws);  // Send a request once the WebSocket is open
