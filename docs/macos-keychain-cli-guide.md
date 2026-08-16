# Managing API Keys and Passwords via macOS Keychain CLI

You can use the built-in macOS `security` CLI tool to store, retrieve, and delete API keys and passwords securely. This prevents you from hardcoding sensitive credentials into your scripts or configuration files.

## 📥 1. Storing an API Key or Password

An API key is treated as a **generic password**. Use the `add-generic-password` subcommand.

### Standard Command

```bash
security add-generic-password -a "$USER" -s "ServiceName" -w "YourSecretKeyOrPassword"
```

### Parameter Breakdown

* `-a` (Account): The username or owner of the credential. Using the system variable `$USER` is standard.
* `-s` (Service): A unique identifier or name for the API/service (e.g., `OpenAI_API_Key`, `GitHub_Token`).
* `-w` (Password): The actual secret string, token, or password.

### 🔐 Secure Alternative (Avoids Shell History)

Including `-w "secret"` leaves your API key in your shell history. To prevent this, leave `-w` empty at the very end of the command to be securely prompted for it:

```bash
security add-generic-password -a "$USER" -s "ServiceName" -w
```

---

## 📤 2. Retrieving and Using the API Key

You can read the password back and inject it directly into environment variables or scripts.

### Read directly to terminal output

```bash
security find-generic-password -s "ServiceName" -w
```

### Load into an Environment Variable (Bash / Zsh)

Add this to your script or your `.zshrc` / `.bash_profile` to load the key seamlessly:

```bash
export SERVICE_API_KEY=$(security find-generic-password -s "ServiceName" -w)
```

*Note: macOS will show a GUI prompt the first time you run this to grant Terminal permission to access that specific keychain item.*

---

## ❌ 3. Deleting a Stored Credential

If you need to rotate your API key or remove it completely, target it by its service name:

```bash
security delete-generic-password -s "ServiceName"
```

---

## 💡 Quick Tips

* **Checking Existing Keys:** You can see your list of generic passwords using `security dump-keychain`, though searching by service name (`-s`) is much faster.
* **Overwriting:** If you run `add-generic-password` for a service that already exists, the command will fail. You must delete the old one first, or use the `-U` flag to update an existing entry.
