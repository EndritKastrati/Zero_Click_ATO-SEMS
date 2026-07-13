<p align="center">
  <a href="https://uni-pr.edu/">
    <img src="/images/logo_up.png" alt="Logo" height="40">
  </a>

<h2 align="center">Universiteti i Prishtines (SEMS) Zero-Click Account Takeover Vulnerability</h3>
</p>

## Description 

During legal security testing of Universiteti i Prishtines “Hasan Prishtina”, main platform: SEMS (Sistemi Elektronik i Menaxhimit te Studenteve). A critical vulnerability was discovered allowing for complete, zero-click professor’s account takeover by chaining an arbitrary email change with the application's forgot-password recovery mechanism. Specifically, the profile update function **POST /Profile/EditProfile** fails to properly validate identity before updating an authenticated user's email address. By manipulating request parameters, an attacker can silently change a victim's registered email address to one email which is under the attacker's control. 

Once the email modification is successfully executed without the victim's knowledge or interaction, the attacker can leverage the standard "Forgot Password" feature. Because the account's primary email has been altered, the password reset token is routed directly to the attacker, allowing them to reset the credentials and gain full control of the account. This represents a critical failure in state-changing transaction verification and account lifecycle security, posing a severe risk to user data integrity and confidentiality. 
<br>
<br>

## Impact 

The successful exploitation of this vulnerability has critical consequences for the institution's Students Electronic Management System (SEMS) and the university community as a whole. Although the proof-of-concept (PoC) will be demonstrated below on a specific faculty account, the underlying systemic flaw extends to the entire user base of over ~158,000 students, professors, and administrative staff. Impact Includes: 

- **Compromise of Academic and Institutional Integrity:** Unauthorized access to a professor or staff account allows an attacker to manipulate final grades, modify exam schedules, alter course enrollments, and tamper with official academic records. This directly undermines the main function and trust and validity of the institution's grading and evaluation systems. 

- **Widespread Data Breach and PII Exposure:** Educational management platforms host extensive Personally Identifiable Information (PII), including national ID numbers, contact details, financial/tuition records, and employment histories. A systemic account takeover capability exposes this sensitive data for thousands of individuals, violating data privacy regulations. 

- **Privilege Escalation and Lateral Movement:** Compromising staff or faculty accounts can serve as a first-step for further attacks. Attackers can utilize internal communication channels to launch highly targeted phishing campaigns against higher-privileged administrative accounts or system administrators. 

- **Severe Operational Disruption:** Mass or targeted account takeovers can lock legitimate users out of essential academic workflows, halting enrollment periods, grading deadlines, and daily university operations, resulting in significant administrative overhead to restore and verify account integrity. 
<br>

## Remediation 

In order to remediate this issue, the following is recommended: 

   - **Require re-authentication for sensitive account changes:** Enforce the validation of the user's current password or a multi-factor authentication (MFA) challenge prior to updating critical account fields like email addresses or phone numbers. 

   - Hide all endpoints which **discover/leak** Professor IDs and related information. 

   - **Validate email ownership before updating:** Utilize temporary, cryptographically secure confirmation tokens sent to the new email address to verify ownership before committing the change to the database. 

   - **Implement dual-channel notifications:** Send immediate automated alerts to both the old and new email addresses whenever a change is requested, ensuring the user is notified of potential unauthorized activity. 

   - **Ask users for their password before changing key personal information:** Always require the user to re-enter their current password before they are allowed to update critical information like their account email or password.
<br>

## Reproduce |  Proof-of-Concept (PoC) 

- In order to reproduce this issue, navigate to SEMS login URL and authenticate using any valid student account: 

   - https://sems.uni-pr.edu/Account/Login 

<div style="text-align: center; margin: 20px 0;">
  <img src="/images/f1.png" alt="Figure 1" style="max-width: 100%; height: auto; border: 1px solid #ddd; border-radius: 8px;">
  <p style="text-align: center; margin: 20px 0;"><strong>Figure 1:</strong> Authenticating using any Student User Account</p>
</div>

- Next, while intercepting browser traffic using a proxy tool (in our case Burp Suite) navigate to the following endpoint and click the “Modify” button:. 

   - https://sems.uni-pr.edu/profile 

<div style="text-align: center; margin: 20px 0;">
  <img src="/images/f2.png" alt="Figure 2" style="max-width: 100%; height: auto; border: 1px solid #ddd; border-radius: 8px;">
  <p><strong>Figure 2:</strong> Intercepting the POST request responsible for profile edit</p>
</div>

- In Burp Suite, right click the intercepted _POST /Profile/EditProfile_ request and select Send to Repeater (or use the shortcut CTRL + R). 

<div style="text-align: center; margin: 20px 0;">
  <img src="/images/f3.png" alt="Figure 3" style="max-width: 100%; height: auto; border: 1px solid #ddd; border-radius: 8px;">
  <p><strong>Figure 3:</strong> Sending the intercepted request to Burp Repeater</p>
</div>

- In the Repeater tab, observe the 2 vulnerable ID parameters used for this vulnerability: 

   - PerdoruesiId - 5/6 digits value, which any SEMS user account has. 

   - User (ID) - Personal University of Prishtina ID number (Students have 12 integers long ID and Professors and Staff 5 integer characters). 

Also observe the “Email” and “GroupId” parameters which will be used for this vulnerability. 

- Now in order for this vulnerability to exist, enumeration and discovery of Professors “PerdoruesiId” and “UserId” were performed and found in the following request: 

   - https://sems.uni-pr.edu/KonfigurimiTemesStudenti 

      - GET /KonfigurimiTemesStudenti/Temat?ProfesoriId=<int> 

   - GET /KerkesaPerdoruesi/GetPerdoruesit 

   - GET /NotimiFleteparaqitjet/Profesoret1 

- With the successful enumeration of the two required parameters, the vulnerability can be fully demonstrated with a full zero-click Account Takeover (ATO). 

   - For the purposes of this demonstration, a legacy faculty account belonging to the late Professor **Nexhat Daci** was utilized as the target to ensure active users and ongoing university operations were not disrupted. 

- In Burp Repeater, within the _POST /Profile/EditProfile_ request performed earlier using the Student account, change the following parameters to match the victim's identifiers and click Send to arbitrarily change the main email address of the victim account. 

      - PerdoruesiId -> 67585 

      - User -> 11138 

      - GrupiId -> 5 

      - Email -> [ Any Email we control ] 

<div style="text-align: center; margin: 20px 0;">
  <img src="/images/f4.png" alt="Figure 4" style="max-width: 100%; height: auto; border: 1px solid #ddd; border-radius: 8px;">
  <p><strong>Figure 4:</strong> Unauthorized profile update targeting the victim account</p>
</div> 

- After clicking Send, observe the 302-Found (and after Follow Redirection the 200-OK) response messages, which indicates that the change was successfully done. 

- Now, return to the SEMS login page and submit a password reset request through the - 

- "Forgot Your Password" feature: https://sems.uni pr.edu/ResetPassword 

<div style="text-align: center; margin: 20px 0;">
  <img src="/images/f5.png" alt="Figure 5" style="max-width: 100%; height: auto; border: 1px solid #ddd; border-radius: 8px;">
  <p><strong>Figure 5:</strong> Submitting a password reset for the victim account</p>
</div>

- Observe the successful message indicating that the link for password-reset was sent to our email address: 

<div style="text-align: center; margin: 20px 0;">
  <img src="/images/f6.png" alt="Figure 6" style="max-width: 100%; height: auto; border: 1px solid #ddd; border-radius: 8px;">
  <p><strong>Figure 6:</strong> Success message — reset link sent to attacker-controlled email</p>
</div> 

- Now in our email, we can see that the email with the link to change the password was sent and we can successfully reset it to confirm the vulnerability. 

<div style="text-align: center; margin: 20px 0;">
  <img src="/images/f7.png" alt="Figure 7" style="max-width: 100%; height: auto; border: 1px solid #ddd; border-radius: 8px;">
  <p><strong>Figure 7:</strong> Reset link received in attacker-controlled inbox</p>
</div>

- Lastly, after successfully resetting the password. We can login and impersonate our victim account. _[ For testing Purposes the Password set was: Testi123. ]_ 

<div style="text-align: center; margin: 20px 0;">
  <img src="/images/f8.png" alt="Figure 8" style="max-width: 100%; height: auto; border: 1px solid #ddd; border-radius: 8px;">
  <p><strong>Figure 8:</strong> Successful login as the victim (Zero-Click Account Takeover confirmed)</p>
</div> 
<br>

## Severity 

This vulnerability is classified as **Critical** severity/rating with a score of **9.9** (from 1-10) due to the following metric breakdown ( _CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H_ ) 

   - **Attack Vector: Network (AV:N):** The exploit can be executed remotely over the internet without requiring physical or local network access. 

   - **Attack Complexity: Low (AC:L):** No specialized conditions or complex configurations are required to trigger the flaw; standard parameter manipulation is sufficient. 

   - **Privileges Required: Low (PR:L):** The attack only requires basic student-level authentication to access the vulnerable endpoint. 

   - **User Interaction: None (UI:N):** The takeover is fully "zero-click"-the victim does not need to click a link, log in, or interact with the application for the attack to succeed. 

   - **Scope: Changed (S:C):** By taking over a higher-privileged or administrative account from a lower-privileged student session, the attacker crosses authorization boundaries, allowing them to impact resources managed by entirely different security contexts. 

   - **Impact Metrics (C:H/I:H/A:H):** The vulnerability leads to a total compromise of **Confidentiality** , **Integrity** , and **Availability** , as gaining administrative or faculty access permits full control over institutional data, system configurations, and grading functionalities. 
<br>

## Concluding Comments 

As both a penetration tester and a student at the University of Prishtina, my primary objective in conducting this assessment is to proactively identify and help remediate security flaws before they can be exploited by malicious attackers.
Securing the SEMS platform is vital to protecting the academic integrity, administrative continuity, and private data of over 158,000 peers, professors, and staff members who rely on this infrastructure daily.
<br>
<br>

    ATTENTION: This report is provided exclusively to the University of Prishtina’s information technology and security administration for remediation purposes. No details of this vulnerability will be disclosed publicly until a proper fix has been implemented and verified.
