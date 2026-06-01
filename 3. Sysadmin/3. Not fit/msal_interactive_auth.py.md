import os
import msal

CLIENT_ID = os.environ.get("MSAL_CLIENT_ID", "your-client-id-here")
TENANT_ID = "common"
AUTHORITY = f"https://login.microsoftonline.com/{TENANT_ID}"
SCOPES = ["Notes.Read"]

app = msal.PublicClientApplication(CLIENT_ID, authority=AUTHORITY)
result = app.acquire_token_interactive(scopes=SCOPES)
print(result)