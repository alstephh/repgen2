My first taks was to implement a new flag on the "report-generator" tool. This
flag (-s/--scoretype) should handle different Severity Score from **CVSS** which is
the standard on this tool. I have made my own researchs about **CVSS** issues and why
is not always ideal before find a different score type. **CVSS** is an industry standard
maintained by NVD which define vulnerability as :

*"A weakness in the computational logic (e.g., code) 
fouund in software and hardware components that, when exploited, results in a negative impact to confidentiality, 
integrity, or availability. Mitigation of the vulnerabilities in this context typically involves coding changes, 
but could also include specification changes or even specification deprecations (e.g., removal of affected protocols 
or functionality in their entirety)."*

The point of scoring vulnerability is to prioritize patching process, obviously based on the *"danger"* (yes, this
word needs more context to be defined but allow me to use it out of context for the sake of discussion) of each vulnerability.
**CVSS is not a Risk Score** but a snapshot in time on how the vulnerability is in a specific software/platform/OS/ecc..
This create a small but focal gap to reflect reality of things, when a vulnerability is patched the CVSS score will
decrease even if the patcher entity grade it as higher (see CVE-2023-35352 affecting Microsoft RDP, CVSS score of 6.5
but CRITICAL for Microsoft). Moreover is a quantitative approach used to prioritize the patching checklist (in the case of Microsoft RDP vuln,
using CVSS would delay the fix even the high rate of threat) and cannot be ideal to client with lack of knowledge aout how 
CVSS score works, looking just at the numbers and create a leaderboard based solely on them. Lastly the context is not properly involved
in the classification, every network is different and a CVSS of 9/10 in an enviroment could rely with proper detection/prevention technique or
in an unimportant assets (in term of security/CIA triad) making the exploit of the vuln not that dangerous as the score rapresentation. 
With "context" we mean not just the victim environment but how much the vuln is exploited in the wild or existence of a public PoC, 
this should be considered into the evaluation to help clients to improve the security posture.

I have looked through some alternatives and I decide to use one which looks pretty interesting, this doesn't mean is ideal for the industry
use but I wanted to implement something different and new. I decided to use SSVC (CVSS but the opposite way), a decision tree apporach which result
is a categorical variable called **Decision**. Stakeholder Specific Vulnerability Categorization leverage the vulnerability prioritizing methods
involving stakeholders involved during th patching process, this new side introduction alone make it really different from the simple CVSS number.
The results is based on decision tree which can be used for every gap found into the victim environment and should take care of the big picture not
just the vulnerability itself. This would mark the same vulnerability inside *hMail* different for equal vuln inside *OutLook*, taking into account
how the vulnerable assets is used inside the client *"mission"*. I used the [CISA SSVC calculator](https://www.cisa.gov/ssvc-calculator) because is simple
and concise, the Decision Tree can be summarized like it follows:

* **State of Exploitation** = present state of the vuln (not future prediction); *None*, *PoC* or *Active*
* **Automable** = difficult and speed of exploitation and consequent actions; *Yes* or *No*
* **Techical Impact** = can be compared on CVSS severity score, how much control the Threat get after exploitation; *Partial* or *Total*
* **Mission & Well-Being** = the sum of 2 categorical variable (Mission Prevelance + Well-Being Impact); *Low*, *Medium* or *High*

The full information can be found [here](https://www.cisa.gov/sites/default/files/publications/cisa-ssvc-guide%20508c.pdf) on how the decision tree work.
Obviously different trees can be developed (because stakeholder change) so SSVC is not a one-shot solution itself and ad-hoc decision tree could be created
for specific type of clients (private company, public institution, critical infrastructure, ecc..). The peculiarity of such system is the results at the end
of the tree, a category not a number. It would not just prioritize but also suggest how to behave on every vulnerabiliy found, you could make a checklist and
work concurently on every issues with an horizontal approach (with proper attention) for everyone inside the patching process:

* **Track** = No action but track the vulnerability situation. Consider to change your approach based on the new informations (for instance a public PoC or APT use)
* **Track+** = Like *Track* but monitor it closely and be prepared for future changes
* **Attend** = Actively request info about this vulnerability and assistance as well, notify (at least) internals about it giving the right attention. Actions should be done
* **Act** = Every stakeholder must be notified in order to meet and plan short-term mitigation/prevention technique.

The SSVC CISA score is similar to the CVSS with obviously no numbers but **Decison Value**, in the *issues files* have been saved like the following under the *severity section* :

```
Track
SSVCv2/E:P/A:Y/T:T/P:M/B:M/M:L/D:T
```
Regarding the challenge itself, working with categorical data forced me to make important addition (no copy-&-paste with parameter changes) on the whole repository (all python and latex files have been touched).
I have explored the whole workflow of the script and learn some python usage new for me (Jinja2 was not that easy to understand at the start), I tried to change just the essential
part of the repository to maintaint the identity of the codes and recognize immediately the changes and how they melt with pre-exisisting task. Same thing applied for the template and style
of final PDF.

(Note that I have manually changed some SSVC string manually to have it all of the same length, this is because different path in the tree may return more value insidethe SVVC string 
so consider this as a beta. I worked under an assumption that a standardize tree have been implemented so all SSVC string have same length making copy-paste enough)


