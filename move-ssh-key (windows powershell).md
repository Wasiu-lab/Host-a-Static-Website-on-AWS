# Windows (PowerShell or Command Prompt)

# Move the key to the Home directory
move "$HOME\Downloads\dev-keypair.pem" "$HOME\dev-keypair.pem"

# Verify if the key exists in the Home directory
Test-Path "$HOME\dev-keypair.pem"