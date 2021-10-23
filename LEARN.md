# Patents On Solana
Solana uses Rust as the basic programming language. If you don’t know Rust already, refer to the Rust for Solana Quest (). It is a great primer to all the concepts and basics of rust you need to know to get started with writing code for Solana programs.

You also need to have a command line understanding. We’ll be operating heavily on the commandline to compile and deploy Solana programs. We recommend using Linux, Mac or WSL if you are on windows.

You must have Node installed on your machine/WSL. If you don’t, you can head over to the following link to install : [https://nodejs.org/en/download/](https://nodejs.org/en/download/)

With these installed we are good to go!

We will be building a Patent Platform on Solana which will allow people to save their idea on Solana without having to make it public (initially storing idea’s hash only), and when the author is ready to make it public he could make it visible to all (this is known as a commitment scheme).

Now on to writing some code..
## Setting Up project and initializing
Assuming you have seen the rust tutorial, now to get things started initialize a new project by typing the command in your preferred directory

*cargo new Patents --bin*

This will create a directory structure as follows:-

	*Patents*

*		|*

*		----> src*

*			|*

*			-------> main.rs*

*		|*

*		-----> Cargo.toml*

 

Rename *main.rs* to *lib.rs*, and delete all its contents this is the file where we will be writing all our rust code.
## Defining The Structure
We have to store a few things on the blockchain, the publickey of the owner, idea’s hash, idea, and an is_public flag.

The structure will be as follows:-

__pub struct Patent {__

__    pub property_owner: String,__

__    pub hash: String,__

__    pub idea: String,__

__    pub is_public: String,__

__}__

As evident property_owner field will store the owner, hash will store the hash of the idea when it is not public, is_public flag will tell if the idea has been made public or not, if yes then the idea will be stored in the idea field.

Next we will be writing logic for the program
## Writing Logic
Now that we have defined our structure we need the login that will interact with this structure and will insert/edit data.

Every program has an entrypoint function whose declaration we will see in later parts, now let us first write the function. This function will perform functions such as creating a new Patent with an idea's hash, secondly making the idea public, all according to the instructions supplied.

__pub fn process_instruction(__

__    program_id: &Pubkey,__

__    accounts: &\[AccountInfo\],__

__    data: &\[u8\],__

__) -> ProgramResult {   }__

The program_id is the identifier for the program itself. When you want to call a program, you must also pass this id, so that solana knows which program is to be executed.

The second parameter to this function is an array of accounts that will be operated upon in this program.

Every user on Solana has a unique account. Every code that you write and deploy also has a unique account. An account can store money in a currency called Lamports. We can transfer lamports from one account to another. When you send a user some money, they can spend it however they want. If you send money to the account operated by a program, that program can use that money defined in the logic of the code itself.

So, coming back, the second parameter to our process function is the list of accounts that will be operated upon in this code. If the function is called to create a new idea, you need to create a new account for this idea and send that as a parameter. If you are calling this function to make the idea public, the array will contain the account of the idea.

The data is the last parameter. The data tells what instruction to execute and all the data that is required to execute the logic.
## Defining Instructions
All of our code will be handled by the function process_instruction, since many operations may exist, we need a parameter to differentiate between what logic to execute.

Recall that we discussed in the last quest that the data parameter also contains the instruction bit, so we will isolate it from the rest of the data and use it to process relevant code.

__*let (instruction_byte, rest_of_data) = data.split_first().unwrap();*__

Recall the the account parameter received contains all the relevant accounts, first we will convert it into iterable array

__*let accounts_iter = &mut accounts.iter();*__

Now we retrieve the Patent account we wanna work upon which we will be giving it via javascript api (discussed in further quests).

__*let nft_account = next_account_info(accounts_iter)?;*__

Further we will check that the Patent account created was created by owner program to avoid any unauthorised changes in the data by some other person who is not entitled to do it.

__*if nft_account.owner != program_id {*__

__*        msg!("Greeted account does not have the correct program id");*__

__*        return Err(ProgramError::IncorrectProgramId);*__

__*    }*__

Now finally we declare our instructions.

__if  \*instruction_byte == 0{__

__	//create new patent with hash__

__}__

__else if  \*instruction_byte == 1{__

__	//make idea public__

__}__
## Create new Patent
The next account in account_iter array will be the creator of the patent.

__*let nft_account_owner = next_account_info(accounts_iter)?;*__

Next we will retrieve the hash of the idea which in this case will be a string of 64 characters. We will decode it from uint8 array

__*let hash = String::from_utf8(rest_of_data\[..64\].to_vec()).unwrap();*__

Now we will get the idea, since we are only storing hash in the first place the idea here will be a dummy string(here underscores) of exactly the same length as the idea entered, so that we can replace it with the idea once it is made public. You may be having doubts at this place that why we need to store a dummy string, this is because the account we create is programmed to store a fixed amount of data, any change in length will cause a serialization error, it is however possible to change the size once it is declared but it is a very complicated process so we will stick to the dummy string method.

__*let idea = String::from_utf8(rest_of_data\[64..\].to_vec()).unwrap();*__

Now we need to create a Patent object and store it in our ProgramData HashMap. For that we need the key for this Patent.

The key for this Patent needs to be unique.

Every account has a public key and a private key. The public key, also called address, is akin to a username. The private key is something we will look at a little later, but it is similar to the password for this account.

We had earlier seen that an account can store lamports i.e money. But an account can also store data. The account for this patent will store the data that we defined in the struct called Patent..

__*let mut nft_account_data:Patent =*__

__*            try_from_slice_unchecked(&nft_account.data.borrow()).map_err(|err| {*__

__*                msg!("Receiving message as string utf8 failed, {:?}", err);*__

__*                ProgramError::InvalidInstructionData*__

__*            })?;*__

__* nft_account_data.property_owner = nft_account_owner.key.to_string();*__

__*nft_account_data.hash = hash;*__

__*nft_account_data.idea = idea;*__

__*nft_account_data.is_public = "0".to_string();*__

Then we store it back on to the account’s data by serializing it, so that it persists.

     __*   nft_account_data*__

__*            .serialize(&mut &mut nft_account.data.borrow_mut()\[..\])*__

__*            .map_err(|err| {*__

__*                msg!("Receiving message as string utf8 failed, {:?}", err);*__

__*                ProgramError::InvalidInstructionData*__

__*            })?;*__
## Making Idea public
It is similar to what we did above now we just have to replace the idea string by the idea which is supplied by the owner the whole code goes as follows

__*        let mut nft_account_data: Patent =*__

__*            try_from_slice_unchecked(&nft_account.data.borrow()).map_err(|err| {*__

__*                msg!("Receiving message as string utf8 failed, {:?}", err);*__

__*                ProgramError::InvalidInstructionData*__

__*            })?;*__

__*        //decode hash from byte array and store it*__

__*        let idea = String::from_utf8(rest_of_data\[..\].to_vec()).unwrap();*__

__*        nft_account_data.idea = idea;*__

__*        nft_account_data.is_public = "1".to_string();*__

__*        *__

__*        nft_account_data*__

__*            .serialize(&mut &mut nft_account.data.borrow_mut()\[..\])*__

__*            .map_err(|err| {*__

__*                msg!("Receiving message as string utf8 failed, {:?}", err);*__

__*                ProgramError::InvalidInstructionData*__

__*            })?;*__

Now return the function using 

__*Ok(())*__
## Including Libraries
Now that we’ve written some logic, let’s make sure we write all the other boilerplate stuff so that we’re able to actually compile the code.

We’ll import the functions we are using in our logic from a package/crate called solana_program

__*use solana_program::{*__

__*    account_info::{next_account_info, AccountInfo},*__

__*    entrypoint,*__

__*    entrypoint::ProgramResult,*__

__*    msg,*__

__*    program_error::ProgramError,*__

__*    pubkey::Pubkey,*__

__*};*__

We also need to tell the compiler to make our ProgramData data structure to be storable. The way to store this data on to the Solana Persistent Storage, is to serialize it. We’ll use a serializer called the BorchSerializer

__*use borsh::{BorshDeserialize, BorshSerialize};*__

Tell the compiler that the struct ProgramData can be serialized and deserialized using this library by including the following line just above the ProgramData struct.

__*\#\[derive(BorshSerialize, BorshDeserialize, Debug)\]*__

Lastly, define which function is to be called when someone is trying to interact with our program. This we do using the entrypoint function that solana_program exposes.

Recall that we discussed that the entrypoint will be declared afterwards.

__*entrypoint!(process_instruction);*__

So your entire file would look like this:

[https://gist.github.com/MayankMittal1/77444843fb973dc673f9bc9391972332](https://gist.github.com/MayankMittal1/77444843fb973dc673f9bc9391972332)
## Setting Up Solana
For setting up solana and other initial steps follow subquests 1-3 on [QuestBook - Deploying the program on to Solana](https://questb.uk/quest/deploying-the-program-on-to-solana-4745)

Now next we will be writing Cargo.toml file, it consists of the dependencies of our project

[Cargo.toml · GitHub](https://gist.github.com/MayankMittal1/9aa0a128311a5b6b1617e585d098991a)

Now let’s compile the program

__*cargo build-bpf --manifest-path=./Cargo.toml*__

This will create a deployable solana file at targets/deploy/patents.so

Now you can deploy this compiled program using

__*solana program deploy targets/deploy/patents.so*__

In a couple of seconds, it will get deployed and display the program id on the console.

Now that we have written our solana code, we need something to interact with it. Now we will be writing a js api to execute the code on solana.