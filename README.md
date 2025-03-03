# hacktitans
import hashlib
import json
import time
from typing import List, Dict, Any

class Block:
    def __init__(self, index: int, previous_hash: str, timestamp: float, data: Dict[str, Any], nonce: int = 0):
        self.index = index
        self.previous_hash = previous_hash
        self.timestamp = timestamp
        self.data = data  # Stores voting results
        self.nonce = nonce
        self.hash = self.compute_hash()

    def compute_hash(self) -> str:
        block_contents = json.dumps({
            "index": self.index,
            "previous_hash": self.previous_hash,
            "timestamp": self.timestamp,
            "data": self.data,
            "nonce": self.nonce
        }, sort_keys=True)
        return hashlib.sha256(block_contents.encode()).hexdigest()

    def mine_block(self, difficulty: int) -> None:
        target = "0" * difficulty
        while not self.hash.startswith(target):
            self.nonce += 1
            self.hash = self.compute_hash()

class Blockchain:
    def __init__(self, difficulty: int = 4):
        self.chain: List[Block] = [self.create_genesis_block()]
        self.difficulty = difficulty

    def create_genesis_block(self) -> Block:
        return Block(0, "0", time.time(), {"message": "Genesis Block"})

    def get_latest_block(self) -> Block:
        return self.chain[-1]

    def add_block(self, data: Dict[str, Any]) -> None:
        latest_block = self.get_latest_block()
        new_block = Block(len(self.chain), latest_block.hash, time.time(), data)
        new_block.mine_block(self.difficulty)
        self.chain.append(new_block)

    def validate_chain(self) -> bool:
        for i in range(1, len(self.chain)):
            current, previous = self.chain[i], self.chain[i - 1]
            if current.hash != current.compute_hash():
                return False
            if current.previous_hash != previous.hash:
                return False
        return True

# Example usage
if __name__ == "__main__":
    voting_chain = Blockchain()
    voting_chain.add_block({"candidate": "Alice", "votes": 150})
    voting_chain.add_block({"candidate": "Bob", "votes": 130})
    
    for block in voting_chain.chain:
        print(f"Index: {block.index}")
        print(f"Previous Hash: {block.previous_hash}")
        print(f"Timestamp: {block.timestamp}")
        print(f"Data: {block.data}")
        print(f"Hash: {block.hash}")
        print("-" * 40)
