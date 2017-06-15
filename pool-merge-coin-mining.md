# Overview
Merge mining allows to **mine one blockchain** but still **get rewarded as if we were mining 2 blockchains simultaneously**.
It works by mining only one blockchain but submitting the results to 2 blockchains. For this to work, it requires one blockchain to trust and rely on the other blockchain to check valid proof-of-works.

The chain actually being mined is called the **parent (PRT) chain**, while the other chain is called the **auxillary (AUX) chain**. For the parent chain, there is almost no difference between a block mined the regular way and a block mined as part of a merged coin mining process. That's why there is no need to modify the code of the daemon of the parent blockchain. For the auxillary chain, a block mined the normal way (i.e a *regular block*) will be accepted with the original code. However, a block mined as a part of a merged coin mining process (i.e a *modified block*) will require a modification of the code of the AUX daemon.

The key to understand double mining is that a *modified block* in the AUX chain will not look valid to a AUX daemon that only knows how to process regular blocks. To accept a modified block, the AUX daemon must rely on some API calls to the PRT chain. The AUX daemon needs to make sure that:
1. the *modified block* contains a hash of a valid block on the PTX chain
2. this PTX valid block itself contains a hash to the current  AUX block header 

# AUX Chain vs PRT Chain
The below schema describes how a *modified block* on the AUX blockchan relates to its *parent block* on the PRT blockchain.
```
########################################              ############################################          
#          AUX BLOCKCHAIN              #              #              PRT BLOCKCHAIN              #
########################################              ############################################
# ...                                  #              # ...                                      #
# ------------------------------------ #              # ---------------------------------------- #
# Block x-1                            #              # Block y-1                                #
# ------------------------------------ #              # ---------------------------------------- # 
# Modified Block x                     #              # Block y                                  #
# - Hash of Transactions Merkle Tree -----------------------------------------------             #
#                                      #              #                            ||            #
#                                      #              # - Transactions             \/            #
# - coinbase_txn -----------------------------------------> Coinbase Tx     [Optional Field]     #
#                                      #              #     Tx1                                  #
#                                      #              #     Tx2                                  #
#                                      #              #     ...                                  #
#                                      #              #                                          #
#                                      #              #     ------------------------------------ #   
#                                      #              #     | Block Header                     | #
#                                      #              #     ------------------------------------ #
# - parent_block -----------------------------------------> | - version                        | #   
#                                      #              #     | - previous_block_header_hash     | #
#                                      #              #     | - merkle_root_hash (transactions)| #
#                                      #              #     | - time                           | #
#                                      #              #     | - nBits (difficulty)             | #
#                                      #              #     | - nonce                          | #
#                                      #              #     ------------------------------------ #
#                                      #              #           ||                             #
#                                      #              #           \/                             #
# - block_hash ---------------------------------------------->  Hash of block header             #
#                                      #              #     ...                                  #
# ------------------------------------ #              # ---------------------------------------- #
# Block x+1                            #              # Block y+1                                #
# ------------------------------------ #              # ---------------------------------------- #
# ...                                  #              # ...                                      #
########################################              ############################################


```


# Definitions
 * Parent (PRT) blockchain: The blockchain being actually mined
 * Auxillary (AUX) blockchain: The other blockchain which accept proof-of-works of the parent chain
 * Daemon: also called full-node, or client. It connects to is the central piece of any blockchain. Usually
 * Miner: the software that tries to create new blocks by searching for valid proof-of-works
 * Pool: the software that sits between the Daemon and Miner and distribute works among miners
 * Regular block template: A block template used for single coin mining. Exist on PTX and AUX chain.
 * Modified block templates: A regular block template + other headers. Exists only on the AUX chain.
 
 # Relationships between Miners, Pool and Daemons
 ```
---------------------------------------------------------
| INDIVIDUAL COMPUTERS                                  |
---------------------------------------------------------
|########### ########### ###########      ###########   |
| # MINER 1 # # MINER 2 # # MINER 3 # ... # MINER X #   |
| ########### ########### ###########     ###########   |
---------------------------------------------------------
   /\           /\          /\             /\ 
   ||           ||          ||             ||
   ||           ||          ||             || 
   ||           ||          ||             ||
   \/           \/          \/             \/
---------------------------------------------------------
| CENTRALIZED SERVER                                    |
---------------------------------------------------------
| #########################################             |
| #                 POOL                  #             | 
| #########################################             |
|                    /\                                 |
|                    ||                                 |
|                    ||                                 |
|                    \/                                 |
| #########################################             |
| #                DAEMON                 #             |
| #########################################             |
| /\                                    /\              |
--||------------------------------------||---------------
  ||                                    ||
--||------------------------------------||--------------- 
| ||                                    ||              |
| \/                                    \/              |
| #################       #################             |
| # DAEMON 1      # <---> # DAEMON 2      #             | 
| #################       #################             |
|     /\          |\      /|       /\                   |
|     ||            \    /         ||                   |
|     ||             \  /          ||                   |
|     ||              \/           ||                   |
|     ||             /  \          ||                   |
|     ||            /    \         ||                   |
|     \/          |/       \|      \/                   |
| #################       #################             |
| # DAEMON 2      # <---> # DAEMON n      #             |
| #################       #################             |
|                                                       |
---------------------------------------------------------  
| DECENTRALIZED P2P NETWORK                             |               
---------------------------------------------------------  


```

# Single Coin Mining
## 1. Miner ask for work
```
#########################################
#                MINER                  #
#########################################
  ||                               /\                  
  || 1. Get work                   || 4. Send regular block template
  \/                               ||    + specific extra_nonce per miner   
#########################################  
#              AUX POOL                 # --> Note sure if AUX pool needs to ask for new block template to daemon for all
#########################################     getWork() requests. Maybe it can just send send the current block template if 
  ||                               /\         it still valid (i.e no one found it yet)   
  || 2. Get regular                || 3. Send regular        
  \/    block template             ||    block template                      
#########################################
#            AUX DAEMON                 #
#########################################
```

##  2. Miner submit its hashed blocks
```
#########################################
#                MINER                  #
#########################################
                   ||
                   || 1. Submit hashed blocks
                   \/
#########################################  
#              AUX POOL                 # 
#########################################
||                                    /\
|| 2. Submit hashed blocks            || 3. Accept/Reject Blocks
\/    solved > AUX difficulty level   ||
#########################################
#            AUX DAEMON                 #
#########################################
```

# Merged Coin Mining
## 1. Miner ask for work
```
################################################################################
#                                 MINER                                        #
################################################################################
  ||                                                           /\                  
  || 1. Get work                                               || 4. Send regular parent block template   
  ||                                                           ||    * + extra_nonce specific to miner
  ||                                                           ||    * + merkleTree hash of modified AUX block template
  \/                                                           ||        inserted in optional field of coinbase tx 
################################################################################
#                              MERGED POOL                                     # --> uses transactions of modified AUX block
################################################################################     template to produce merkeTree hash
 ||                     /\                     ||                            /\              
 || 2. Get modified AUX || 3. modified AUX     || 2. Get regular PRT         ||  3. regular PRT    
 \/    block templates  ||    block templates  \/    block templates         ||     block templates                        
#########################################  #####################################
#            AUX DAEMON                 #  #              PRT DAEMON           #
#########################################  #####################################
```

## 2. Miner submit PRT hashed blocks
```
################################################################################
#                                    MINER                                     #
######################################### #######################################
                              ||
                              || 1. Submit PRT hashed blocks
                              \/
################################################################################
#                              MERGED POOL                                     #
################################################################################
                                           ||                                  /\
                                           || 2. Submit PRT hashed blocks      || 3. Accept/Reject Blocks
                                           \/    solved > PRT difficulty level ||
#########################################  #####################################
#            AUX DAEMON                 #  #              PRT DAEMON           #
#########################################  #####################################
```

## 3. Pool Submit AUX ashed blocks (once PRT block has been confirmed by PRT chain)
```
################################################################################
#                                    MINER                                     #
################################################################################
                          

                        
################################################################################
#                              MERGED POOL                                     #
################################################################################
 ||                                    /\
 ||  2. Submit AUX hashed blocks       || 3. Accept/Reject blocks
 \/     solved > AUX difficulty level  ||
#########################################  #####################################
#            AUX DAEMON                 #  #           PRT DAEMON              #
#########################################  #####################################
 ```