**Task 6: complete the stake, redeem, and claim flow (including 6hr unbonding)**	
- Set variable
```
STRIDE_ADDR_NAME=`strided keys list -n`
STRIDE_ADDR=`strided keys show $(strided keys list -n) -a`
STRIDE_CHAIN_ID="STRIDE-TESTNET-2"
GAIA_CHAIN_ID="GAIA"
GAIA_ADDR="cosmos1u0rxg6u76xnlp276gnuymtx45dtfqd3etwvxcn"
```
- Stake IBC
```
strided tx stakeibc liquid-stake 10000000 uatom --chain-id $STRIDE_CHAIN_ID -y --from $STRIDE_ADDR_NAME
```
![image](https://user-images.githubusercontent.com/91453629/183052486-144ecbda-2318-47a9-a582-7d61dba1c411.png)

- Redeem IBC
```
strided tx stakeibc redeem-stake 100000 $GAIA_CHAIN_ID $GAIA_ADDR --chain-id $STRIDE_CHAIN_ID -y --from $STRIDE_ADDR_NAME
```
![image](https://user-images.githubusercontent.com/91453629/183052679-6b9a6010-92ae-4425-8c27-580ddad6e4f1.png)

- Claim IBC
```
strided q records list-user-redemption-record --limit 100 --output json | jq --arg $STRIDE_CHAIN_ID '.UserRedemptionRecord | map(select(.sender==$GAIA_ADDR))'
```
Output command is as below, select sender address and ZONE which has value of **Claimable** equal to **true**
![image](https://user-images.githubusercontent.com/91453629/183052877-4a9ad67c-5666-4b79-bd3a-277d026cd9d3.png)

Do below command to claim which corresponding ZONE, sender address acquired from above command
```
strided tx stakeibc claim-undelegated-tokens $GAIA_CHAIN_ID 413 stride1n3nt4zm9ptt0c5rpp3l67d4qvp3j55wavp8wzf --chain-id $STRIDE_CHAIN_ID --from $STRIDE_ADDR_NAME -y --fees=500ustrd
```
![image](https://user-images.githubusercontent.com/91453629/183053434-36b916f3-0147-41e2-a36a-b539a52c9745.png)

- Check txh from ur wallet in explorer
![image](https://user-images.githubusercontent.com/91453629/183053573-e5e5c1e7-426f-4a59-943f-4480de0de473.png)

====================================================================================  
**Task 7: run a relayer on ICA channels specified in #validator-announcements for at least 7 days**
- Follow below link for setting up Hermes Relayer   
    [HERMES Relayer](https://github.com/viennguyenbkdn/Cosmos_Stride/blob/main/STRIDE-TESTNET-2/Guide%20for%20Hermes%20Relayer.md)
- Follow below link for setting up GO Relayer   
    [GO Relayer](https://github.com/viennguyenbkdn/Cosmos_Stride/blob/main/STRIDE-TESTNET-2/Guide%20for%20v2%20GO%20Relayer.md)
- Team will snapshot, no need to submit form

==================================================================================== 
**Task 8: relay using the new v2 go relayer**   
- Follow below link for setting up GO Relayer   
    [GO Relayer](https://github.com/viennguyenbkdn/Cosmos_Stride/blob/main/STRIDE-TESTNET-2/Guide%20for%20v2%20GO%20Relayer.md)
- Submit form as team required.    
==================================================================================== 
**Task 9: relay interchain queries using the new v2 go relayer **

==================================================================================== 
**Task 10: run validator for at least 7 days (being inactive is OK, it still qualifies)
- Team will snapshot onchain, no need submit
   
