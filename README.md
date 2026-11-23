import os
import json
import base64
import hashlib
from cryptography.fernet import Fernet

VAULT_FILE = "vault.dat"
SALT = b"secure_salt_123"

def derive_key(master_password):
    key = hashlib.pbkdf2_hmac(
        "sha256",
        master_password.encode(),
        SALT,
        100000
    )
    return base64.urlsafe_b64encode(key)

def load_vault(cipher):
    if not os.path.exists(VAULT_FILE):
        return {}

    with open(VAULT_FILE, "rb") as f:
        encrypted_data = f.read()

    try:
        decrypted = cipher.decrypt(encrypted_data)
        return json.loads(decrypted.decode())
    except:
        print("Invalid master password.")
        exit()

def save_vault(data, cipher):
    encrypted = cipher.encrypt(json.dumps(data).encode())
    with open(VAULT_FILE, "wb") as f:
        f.write(encrypted)

def main():
    master_password = input("Enter master password: ")
    key = derive_key(master_password)
    cipher = Fernet(key)

    vault = load_vault(cipher)

    while True:
        print("\n1. Add Password")
        print("2. Get Password")
        print("3. Exit")

        choice = input("Choose option: ")

        if choice == "1":
            service = input("Service name: ")
            username = input("Username: ")
            password = input("Password: ")

            vault[service] = {
                "username": username,
                "password": password
            }

            save_vault(vault, cipher)
            print("Saved successfully.")

        elif choice == "2":
            service = input("Service name: ")

            if service in vault:
                print("Username:", vault[service]["username"])
                print("Password:", vault[service]["password"])
            else:
                print("Service not found.")

        elif choice == "3":
            print("Goodbye 👋")
            break
        else:
            print("Invalid option.")

if __name__ == "__main__":
    main()
