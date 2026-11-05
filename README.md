# Password Manager

A command-line password manager written in Python. Multiple users can create accounts on the same machine, and each user can save, view, edit and delete passwords for their apps. Account passwords are stored as salted and peppered hashes, and saved app passwords are encrypted with [Fernet](https://cryptography.io/en/latest/fernet/) symmetric encryption.

> **This is a learning project.** It explores hashing, salting, peppering and symmetric encryption, but it is not secure enough for real passwords. See [Security notes](#security-notes).

## Features

- **Multi-user accounts**: sign up, sign in, change your account password, or delete your account.
- **Hidden password entry**: passwords are typed via `getpass`, so they don't echo to the terminal.
- **Salted and peppered hashing**: account passwords get a random 16-character salt and a random one-character pepper before hashing.
- **Hashed usernames**: usernames are stored as hashes, not plain text.
- **Encrypted vault**: app names and their passwords are both encrypted, so the stored data doesn't reveal which apps you have accounts on.
- **Friendly prompts**: viewing, editing or deleting an app with no saved password offers to create one; saving a duplicate offers to edit the existing entry.

## Getting started

### Requirements

- Python 3
- Packages listed in `requirements.txt` (`cryptography` and its dependencies)

### Installation

```bash
git clone https://github.com/sahai-aditya/Password-Manager.git
cd Password-Manager
pip install -r requirements.txt
```

It's a good idea to use a virtual environment (`venv/` is already in `.gitignore`):

```bash
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### Run

Run from inside the project folder, since `users.json` and `passwords.json` are opened with relative paths:

```bash
python main.py
```

## Usage

### Sign in / sign up

```
1. Sign In
2. Sign Up

Enter Option Number
>>>
```

- **Sign Up** asks for a username (stored lowercase) and a password, which you type twice to confirm. You're signed in straight away.
- **Sign In** asks for your username and password.
- Enter **q** to quit.

### Once signed in

```
1. View Saved Password
2. Write New Password
3. Edit Old Password
4. Delete Password
5. Change Account Password
6. Delete Account
```

| Option | What it does |
|--------|--------------|
| 1 | Asks for an app name and prints its saved password |
| 2 | Saves a password for a new app |
| 3 | Replaces the saved password for an app |
| 4 | Deletes the saved password for an app |
| 5 | Sets a new account password (asks twice to confirm) |
| 6 | Deletes your account and every password saved under it, then returns to the sign-in menu |

App names are case-insensitive. Enter **q** at the menu to sign out and exit.

## How it works

### Account passwords (`UserManager.py`)

On sign-up:

1. A random 16-character **salt** is generated from `[a-zA-Z0-9]`.
2. A random single-character **pepper** from the same set is chosen and then discarded.
3. The hash of `salt + password + pepper` is stored together with the salt. The username is hashed too.

On sign-in, the pepper isn't known, so the program tries all 62 possible pepper characters and accepts the login if any one of them produces the stored hash.

### Saved app passwords (`PasswordManager.py`)

Each entry gets its own freshly generated Fernet key. The app name and password are both encrypted with that key, and the entry is stored as:

```json
{
  "key": "<fernet key>",
  "app": "<encrypted app name>",
  "password": "<encrypted password>"
}
```

Entries are grouped under each user's hashed username in `passwords.json`. To find an app, each entry is decrypted and compared with the name you typed.

## Project structure

```
Password-Manager/
├── main.py              # Interface: menus and the main loop
├── InputMethods.py      # Prompts for usernames, passwords and app names
├── UserManager.py       # Sign up, sign in, change password, delete account
├── PasswordManager.py   # Save, view, edit, delete encrypted app passwords
├── users.json           # Hashed accounts (starts as [])
├── passwords.json       # Encrypted vault entries per user (starts as [])
└── requirements.txt
```

## Security notes

This project was built to learn the concepts, and it has weaknesses that make it unsuitable for real use:

- **Encryption keys are stored next to the data they protect.** Each Fernet key sits in the same JSON entry as the encrypted password, so anyone who can read `passwords.json` can decrypt everything without knowing any account password. A real password manager derives the encryption key from the user's master password (for example with PBKDF2, scrypt or Argon2) and never stores it.
- **MD5 is used for hashing.** MD5 is fast and broken for security purposes; password hashing should use a slow, purpose-built function such as bcrypt, scrypt or Argon2.
- **The data files are plain JSON in the project folder** and are not ignored by git, so be careful not to commit real entries.
- **The pinned dependency versions are from 2021** and may need updating to install on recent Python versions.
