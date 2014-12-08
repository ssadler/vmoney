VMoney
======

Toy cryptocurrency based on GIT.

Motivation
----------

This is a proof of concept to test the idea that data driven applications can be modelled on top of versioned
immutable data structures and that merge semantics can be used meaningfully to synchronize data as well as code.

How it works
------------

* The transaction ledger is contained in the GIT commit history.
* The tree pointed to by the latest commit contains the balance sheet.
* A transaction is a commit that modifies the balance sheet.

There are two types of file in the data folder:

* `data/{address}/balance/{txid}`; spendable outputs; should never conflict.
* `data/{address}/tx`; address's last transaction details, conflicts on double spend.

**Transaction flow**

1. To perform a transaction, a user must have some spendable inputs under his/her control. 
2. A transaction ID is generated by hashing the input parameters (including the txids of the inputs that it spends).
3. The spent inputs are deleted from the GIT tree.
4. An output is created under the recipient's balance folder.
5. A change output may also be created under the sender's balance folder.
6. The sender's tx file is updated with the tx parameters and signature.

Example: https://github.com/alphanode/vmoney/commit/67500d1

Q & A
-----

**How does one use it to send money?**

A transaction commit is made with the command line tool, then `git push`ed it to the recipient. 

**What about double spends?**

When the same inputs are spent twice, a conflict will occur on the `tx` file when the two recipients try to merge
their repositories. This becasue `tx` files must be updated in correct sequence. The rest of the files in the data/
folder, in contrast, should never conflict, because they have unique paths and they are never mutated, just created
once and then deleted when spent.

**What if the recipients never merge their branches?**

It's up to the recipients to check that received transactions have been propagated throughout the network.
This is a service that could be provided by trusted third parties.
In Bitcoin, the blockchain performs this function and requires no trust (in theory).

**How does it check that a transaction is valid?**

All transactions (balance changing commits) must conform to a protocol in order to be valid.
There are 2 kinds of validation done on a transaction:

1. Cryptographic verification, to verify that the sender owns the address and has signed the transaction.
2. Data validation, to verify that the correct changes have been made to the balance sheet. This is achieved by
   replaying the transaction against the parent commit's tree, using the parameters from the `tx` file, then
   comparing the resulting tree with the one from the transaction commit.

Layout
------

    vmoney.py         # Code
    data/             # Data folder
        V7Mm1Bhsxk8/  # Resources belonging to an address
            tx        # Last transaction file, conflicts on merge of double spend
            balance/  # Folder to contain spendable inputs named after tx id


Setup
-----

On Debian based linux:

    sudo apt-get install libssl-dev libffi-dev libgit2-dev python-pip python-virtualenv

Then:

    virtualenv .env
    source .env/bin/activate
    pip install -r requirements.txt
    ./vmoney.py balance
    # Mint some bits
    ./vmoney send --mint 1000 {recipient}
