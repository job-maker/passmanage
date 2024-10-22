# passmanager
import streamlit as st
from cryptography.fernet import Fernet
import os
import random
import string

# Generate a key if it doesn't exist
def generate_key():
    if not os.path.exists("secret.key"):
        key = Fernet.generate_key()
        with open("secret.key", "wb") as key_file:
            key_file.write(key)

# Load the existing key
def load_key():
    return open("secret.key", "rb").read()

# Encrypt the password
def encrypt_password(password, key):
    f = Fernet(key)
    return f.encrypt(password.encode()).decode()

# Decrypt the password
def decrypt_password(encrypted_password, key):
    f = Fernet(key)
    return f.decrypt(encrypted_password.encode()).decode()

# Save password to file
def save_password(site, username, password):
    with open("passwords.txt", "a") as f:
        f.write(f"{site} | {username} | {password}\n")

# Read stored passwords
def load_passwords():
    if os.path.exists("passwords.txt"):
        with open("passwords.txt", "r") as f:
            return f.readlines()
    return []

# Password Generator
def generate_random_password(length):
    characters = string.ascii_letters + string.digits + string.punctuation
    return ''.join(random.choice(characters) for _ in range(length))

# UI using Streamlit
def main():
    st.title("🔒 Password Manager with Generator")

    # Generate or load the encryption key
    generate_key()
    key = load_key()

    # Tabs for navigation
    tabs = ["Store Password", "View Passwords", "Generate Password"]
    choice = st.sidebar.selectbox("Choose Option", tabs)

    if choice == "Store Password":
        st.subheader("Store a New Password")

        site = st.text_input("Site Name", placeholder="e.g., gmail.com")
        username = st.text_input("Username", placeholder="e.g., johndoe")
        password = st.text_input("Password", type="password", placeholder="Enter the password")

        if st.button("Save Password"):
            if site and username and password:
                encrypted_password = encrypt_password(password, key)
                save_password(site, username, encrypted_password)
                st.success(f"Password for {site} saved successfully!")
            else:
                st.error("Please fill in all fields.")

    elif choice == "View Passwords":
        st.subheader("View Stored Passwords")

        passwords = load_passwords()
        if passwords:
            for entry in passwords:
                site, username, encrypted_password = entry.strip().split(" | ")
                decrypted_password = decrypt_password(encrypted_password, key)
                st.write(f"**Site**: {site} | **Username**: {username} | **Password**: {decrypted_password}")
        else:
            st.info("No passwords stored yet.")

    elif choice == "Generate Password":
        st.subheader("Generate a Strong Password")
        password_length = st.slider("Password Length", 8, 32, 12)
        
        if st.button("Generate Password"):
            generated_password = generate_random_password(password_length)
            st.write(f"**Generated Password**: `{generated_password}`")
            st.write("You can now store this password using the 'Store Password' tab.")

if __name__ == '__main__':
    main()
