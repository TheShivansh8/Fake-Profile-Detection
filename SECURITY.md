# Repository Security

## Prohibited content

Never commit, attach or paste:

- Passwords, OTPs, recovery codes or remote-access PINs
- API keys, tokens, private keys, connection strings or real `.env` files
- PAN, Aadhaar, passport or other identity-document details
- Bank statements, payslips, salary records or financial information
- Medical/disability documents or unnecessary personal information
- Complete residential address or background-verification documents
- Confidential company/client data, production exports or unauthorised source code

## Approved content

- Non-confidential intern ID, name, batch, track, shift and joining date
- Learning plans and approved resource links
- Daily work descriptions and repository evidence
- Sanitised technical errors and screenshots
- Approved sample/test data
- Project source and documentation authorised for this repository

## Before every push

- [ ] Review `git status`.
- [ ] Review staged changes with `git diff --staged`.
- [ ] Confirm no secret/configuration file is staged.
- [ ] Confirm screenshots/logs are sanitised.
- [ ] Confirm the target branch and repository are correct.

## Security incident

If confidential information or a credential is exposed:

1. Stop using or sharing it.
2. Privately notify the authorised security/coordinator contact immediately.
3. Rotate or revoke credentials through the authorised administrator/account owner.
4. Do not assume deleting the latest file/commit removes it from Git history.
5. Follow the authorised remediation procedure.

Do not open a public issue containing the exposed value.
