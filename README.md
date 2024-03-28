# Blockchain-based-Copyright-Management
開発ブロックチェーンベースの著作権管理システムにより、クリエイターは自分の作品を保護し、ロイヤリティを効率的に収集することができます。
from web3 import Web3
from solcx import compile_standard, install_solc

# Connect to an Ethereum testnet
w3 = Web3(Web3.HTTPProvider('http://localhost:8545'))
w3.eth.defaultAccount = w3.eth.accounts[0]

# Compile the Solidity contract
install_solc('0.8.0')
with open('CopyrightManagement.sol', 'r') as file:
    copyright_contract_source = file.read()

compiled_sol = compile_standard({
    "language": "Solidity",
    "sources": {"CopyrightManagement.sol": {"content": copyright_contract_source}},
    "settings": {"outputSelection": {"*": {"*": ["abi", "metadata", "evm.bytecode", "evm.bytecode.sourceMap"]}}}
})

# Deploy the contract
abi = compiled_sol['contracts']['CopyrightManagement.sol']['CopyrightManagement']['abi']
bytecode = compiled_sol['contracts']['CopyrightManagement.sol']['CopyrightManagement']['evm']['bytecode']['object']
CopyrightManagement = w3.eth.contract(abi=abi, bytecode=bytecode)
tx_hash = CopyrightManagement.constructor().transact()
tx_receipt = w3.eth.waitForTransactionReceipt(tx_hash)
contract = w3.eth.contract(address=tx_receipt.contractAddress, abi=abi)

# Functions to interact with the contract
def register_work(title):
    tx_hash = contract.functions.registerWork(title).transact()
    w3.eth.waitForTransactionReceipt(tx_hash)

def verify_ownership(creator, title):
    return contract.functions.verifyOwnership(creator, title).call()

# Example usage
register_work("My Artwork")
print(verify_ownership(w3.eth.accounts[0], "My Artwork"))
