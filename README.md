# Linux Identity Lifecycle Lab

I built this lab to practice managing access when someone joins a company, changes departments, or leaves. I used Linux accounts and groups to control access and Wazuh to investigate a failed login.

## Setup

- VirtualBox on Windows
- Ubuntu-Target for accounts, groups, and department folders
- Wazuh-Server for monitoring
- Kali for SSH access and the Wazuh dashboard
- Private network: 192.168.50.0/24

This builds on my [SSH threat detection lab](https://github.com/JCLopez14/ssh-threat-detection-lab).

## Joiner: Granting access

I created a test employee named `lab-alex` and added the account to `lab-finance`. I created a Finance folder with group ownership and `2770` permissions.

Alex could create and read a file in the folder. Another test account, `lab-jamie`, received Permission denied.

![Finance access validation](screenshots/01-finance-access-validation.png)

## Mover: Changing departments

I removed Alex from Finance and added the account to IT. I tested both folders to confirm Finance access was denied and IT access worked.

![Department move validation](screenshots/02-department-move-validation.png)

## Leaver: Removing access

I expired Alex's account, locked its password, and removed IT membership. I confirmed department access was denied and no processes were running under the account.

I also tested a local login using `su`, which returned Authentication failure.

![Offboarding account checks](screenshots/03-offboarding-account-and-access.png)

![Local login denied](screenshots/04-offboarding-local-login-denied.png)

## Wazuh investigation

Wazuh recorded the failed login attempt on Ubuntu-Target.

- Initiating account: `vboxuser`
- Target account: `lab-alex`
- Program: `su`
- Rule ID: `5503`
- Description: PAM: User login failed

These two screenshots show different sections of the same event.

![Wazuh login event](screenshots/05a-wazuh-offboarding-login-event.png)

![Wazuh rule details](screenshots/05b-wazuh-offboarding-rule-details.png)

## Final access review

| Check | Result |
|---|---|
| Alex | Account expired, password locked, and department memberships removed |
| Jamie | No Finance or IT membership |
| Department folders | Correct group ownership and restricted permissions |

![Final access review](screenshots/06-final-access-review.png)

## What I learned

I practiced creating users, assigning access, changing department memberships, and offboarding an account. I checked each change with access tests and reviewed the failed login in Wazuh.

I also learned that locking a password alone does not reliably block SSH keys. Existing sessions can keep old group memberships until they end.

## Limitations

This was a manual Linux lab with fictional employees. I did not use SailPoint ISC or automate provisioning.

The Wazuh event confirms authentication failed, but does not prove account expiration caused the failure. SSH key login denial for Alex was not tested.