# HTML Fund Me

Blockchain Course Lesson 8 (vid 12:32:57). [Vid](https://www.youtube.com/watch?v=gyMwXuJrbJQ) and [repo](https://github.com/smartcontractkit/full-blockchain-solidity-course-js).

Creating a basic frontend to interact with our contract.

I made notes as I went along in `NOTES.md`.

## ERRORS

1. There is an error message in the terminal running the hardhat node when ETH is sent from Metamask to the contract. It says 'You need to spend more ETH!'.

   **NOTE**: The transaction does go through though so it is technically working.

1. The withdraw function doesn't work, the Metamask should pop up to confirm the transaction but doesn't, and theres an error in the browser console.

   **FIXED** when importing an account into Metamask from the hardhat node use the first one ie. index 0. This is the account that is the owner of the contract so only it can withdraw.
