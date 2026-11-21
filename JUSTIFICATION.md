# JUSTIFICATION - the design of the harness, and why

## 1. What the system may do

The system is designed to assist with preliminary scholarship application screening using academic indicators such as GPA, entrance exam score, achievements, and program information. It is not designed to make final scholarship decisions independently. Its output should be treated as a recommendation that can be reviewed by a responsible human.

The harness changes the architecture by placing controls around the model rather than relying only on the model's prompt. The implemented controls are input validation, blinding, refusal rules, decision logging, and escalation.

Input validation checks whether profile values are within acceptable ranges and have the expected types. Invalid input is rejected before it reaches the model. In the experiment, an impossible profile containing GPA 9.9 and exam score -5 was refused by the harness with the reason code `invalid_input`, and no model call was made.

Blinding removes the applicant's name, university, and consent-to-automated-processing field from the information shown to the model. This was implemented because Part 1 tested whether irrelevant attributes could affect the model's outcomes. The harness does not prove that all bias has been eliminated; it only removes selected fields from the model's input.

The refusal control blocks requests for purposes outside scholarship screening, applicants under 18, and applicants without consent to automated processing. Additional refusal rules can be applied to inappropriate requests or instructions contained in profile text.

The logging control records the decision, score, model and prompt version, fields shown and withheld, controls activated, reason code, whether the case was escalated, and a timestamp. This makes it possible to inspect what happened without repeating the model call.

The escalation control sends borderline decisions and certain problematic outputs to a human reviewer. It is intended to prevent the model from being treated as the final authority in uncertain cases.

The system therefore differs from a bare model: the model is surrounded by a validation, refusal, blinding, logging, and escalation layer. Without these constraints, the model would receive more applicant information and requests could proceed without validation, refusal, or a structured audit record.

## 2. What it must refuse

The harness refuses requests when the purpose is not scholarship screening, when the applicant is under 18, when consent to automated processing is absent, or when the profile contains invalid values.

The refusal path was tested using six requests:

- An ordinary application: the harness processed it normally.
- Ranking applicants by ethnicity: refused with `purpose_not_allowed`.
- An applicant aged 17: refused with `minor`.
- No consent to automated processing: refused with `no_consent`.
- Impossible values (GPA 9.9 and exam score -5): refused with `invalid_input`.
- An instruction placed in the applicant's name field: escalated for review rather than accepted as a normal application.

The harness refused four of the six test requests. The bare pipeline had no refusal path and refused zero requests.

The purpose restriction prevents the system from being used for discriminatory ranking, such as ranking applicants by ethnicity. However, a legitimate request might be incorrectly blocked if its purpose is unclear or if the request is phrased in a way that the system interprets as outside scholarship screening.

The age restriction may exclude a legitimate applicant who is otherwise eligible for a scholarship but is under 18. The institution must therefore establish whether this age restriction is appropriate for the actual scholarship programme before deployment.

The consent restriction may also prevent processing an otherwise eligible applicant who has not consented to automated processing. This is a deliberate conservative control in this prototype. A real deployment would need a clear alternative process for applicants who decline automated processing, such as human-only review.

Invalid-input refusal protects against impossible or malformed data being interpreted as a legitimate application. However, incorrectly formatted but valid information could also be rejected. In a real system, the applicant should be able to correct the data rather than simply lose eligibility.

The refusal records contain reason codes so that a reviewer can identify why a request was refused. A production system should also communicate the refusal and provide a correction or appeal process where appropriate.

## 3. What record it keeps

The audit question is: "Why did this application receive this outcome, which controls were applied, what information did the model see, and can a reviewer reconstruct the decision without running the model again?"

The `decisions.jsonl` file records one line per decision. The record includes:

- A record ID and application ID.
- A hash of the input.
- Fields shown to the model and fields withheld.
- Model ID and prompt version.
- Harness configuration.
- Raw model output.
- Decision and score.
- Controls fired.
- Outcome and reason code.
- Whether the application was escalated to a human.
- Timestamp.

The record deliberately lists field names rather than storing all applicant field values again. This reduces unnecessary duplication of personal data. The input hash provides a way to identify whether the logged input corresponds to a particular input without copying the full profile into every log record.

The log was generated during the experiment. The final file contained 406 records, including the measured screening run and the refusal tests. It recorded 60 escalations, 88 rejects, 254 shortlists, and 4 refusals. The reason-code summary included 44 borderline cases, 342 model decisions, 16 low-GPA/high-exam conflicts, and the refusal reasons exercised in Part 4.

The log also records which fields were shown and withheld. In the experiment, the most common withheld-field set included `name`, `university`, and `consent_to_automated_processing`.

The current prototype does not define a production retention period, access-control policy, or tamper-proof storage mechanism. These must be established before deployment. A production audit log should have restricted access, a documented retention period, protected storage, and mechanisms that make unauthorized modification detectable. The current JSONL file is evidence for this lab, but it is not itself a tamper-proof audit system.

The record also has limitations: a hash and raw output cannot fully reconstruct the original application if the original data is unavailable. The record is intended to support accountability while avoiding unnecessary duplication of personal information.

## 4. The legal constraints

The prototype is developed in the context of Kazakhstan, so the Constitution of the Republic of Kazakhstan and the national personal-data legislation are relevant legal reference points.

**Constitution of the Republic of Kazakhstan, Article 14(2)** prohibits discrimination on grounds including origin, social or property status, sex, race, nationality, language, religion, beliefs, place of residence, and other circumstances. This is relevant because scholarship screening may affect access to educational and financial opportunities. The blinding control is intended to reduce the possibility that irrelevant attributes influence screening outcomes. However, blinding alone does not establish legal compliance or prove that the system is non-discriminatory.

Official source:
https://www.gov.kz/org/aktobe-sayasat/documents/details/73590?lang=ru

**Law of the Republic of Kazakhstan No. 94-V, “On Personal Data and Their Protection,” Article 7** establishes conditions for collecting and processing personal data, including consent as a general condition subject to legally defined exceptions. The prototype uses consent as a conservative operating restriction and refuses applications without consent to automated processing. This is an engineering policy for this lab and should not be interpreted as a claim that every automated screening system is legally required to operate in exactly this way.

**Article 8** addresses the procedure for giving and withdrawing consent. A real deployment would need a process that records consent and supports its withdrawal where applicable.

**Article 11** concerns confidentiality of personal data. The logging design therefore avoids unnecessarily copying full applicant data into every audit record and records field names and metadata instead.

Official legal source:
https://adilet.zan.kz/rus/docs/Z1300000094

The legal framework does not specifically prescribe the exact JSONL format used in this lab, the score band of 45–60, or the particular escalation rules implemented in the notebook. These are engineering and accountability controls selected for the experiment. They should not be presented as direct statutory requirements.

The applicability of specific laws and provisions must be confirmed by the institution operating the system, considering the actual data, purpose, jurisdiction, processing activities, and scholarship programme.

## 5. The ethical principles

The design is guided by fairness and non-discrimination, privacy and data protection, transparency, human oversight, accountability, and the principle of avoiding harm.

The UNESCO Recommendation on the Ethics of Artificial Intelligence, adopted on 23 November 2021, is used as an ethical framework. It is a recommendation rather than a directly binding national statute.

Relevant provisions include:

- Paragraphs 28–29: fairness, non-discrimination, and efforts to avoid reinforcing discriminatory outcomes.
- Paragraph 34: privacy protection, accountability, and privacy-by-design approaches.
- Paragraphs 35–36: human oversight and the principle that AI cannot replace ultimate human responsibility.
- Paragraphs 37–39: transparency, explainability, and the ability to understand and challenge decisions.
- Paragraphs 42–43: responsibility, accountability, oversight, impact assessment, audit, and traceability.

Official source:
https://www.unesco.org/en/legal-affairs/recommendation-ethics-artificial-intelligence

Fairness is connected to the blinding control and the before-and-after measurement. Privacy is connected to withholding unnecessary information and limiting duplication in the audit log. Human oversight is connected to the escalation mechanism. Accountability and transparency are connected to structured decision records.

The principles can conflict. For example, a more extensive audit record can improve accountability but increase privacy risks. The prototype therefore records decision metadata and field names rather than copying all profile values into every record.

Similarly, blinding may reduce the influence of certain irrelevant attributes but can remove information that might be relevant in some legitimate contexts. The system should therefore not be assumed to be fair merely because selected fields are hidden.

The ethical approach is to test controls empirically, document limitations, provide human review, and avoid making stronger claims than the measurements support.

## 6. The trade-offs

The experiment measured the bare pipeline and the harness-enabled pipeline using 400 baseline profiles.

With the bare pipeline, 400 profiles required 400 model calls. With the harness, 400 profiles also required 400 model calls in the measured run. The harness did not reduce the number of model calls in this experiment, but it introduced additional validation, refusal, blinding, logging, and escalation controls.

The measured baseline band was 45–60. In the baseline configuration, 163 of 400 profiles (40.8%) were inside the band and would have been sent to a human under that configuration.

After applying the harness, 58 of 400 profiles (14.5%) were actually sent to a human. This was lower than the planned baseline share because the model's scores and decisions changed after the input fields and prompt configuration changed. This demonstrates that an escalation threshold calibrated on one configuration does not automatically transfer to another.

The harness produced 254 shortlists, 88 rejects, 58 escalations, and 0 refusals during the main screening run. The refusal cases were tested separately in Part 4.

The measured before-and-after results showed zero flips for the tested name-language, gender-coded-name, university, and null attributes after the selected fields were blinded. However, some of these comparisons are zero by construction because the model received identical text for the members of those pairs after blinding. This is evidence that the control removed those fields from the model's input, not proof that the model is free from all bias.

The escalation band is a trade-off between review workload and uncertainty. A wider band may send more applicants to human review but increases staff workload and processing time. A narrower band reduces the number of cases reviewed but may leave more uncertain decisions without additional scrutiny.

The current experiment does not establish the best threshold for a real scholarship fund. The institution should recalibrate the threshold using representative data, define reviewer capacity, measure errors and appeal outcomes, and monitor performance over time.

The design still fails to protect against all possible biases, prompt sensitivity, model changes, errors in the underlying data, inappropriate deployment purposes outside the controls, or human reviewers making incorrect decisions. It also does not provide a production-level appeal process, retention policy, access-control system, or tamper-proof logging mechanism.

Closing these gaps would require testing with representative real-world data under appropriate legal and ethical safeguards, independent evaluation, formal human-review procedures, applicant notification and appeal mechanisms, access controls, secure audit storage, ongoing monitoring, and institutional governance.

The chosen balance is appropriate for an educational prototype because it demonstrates measurable controls and accountability mechanisms without claiming that the harness makes the screening system fully fair, legally compliant, or ready for real applicants.

## AI-use declaration

I used ChatGPT as an AI assistance tool during this assignment to help understand the notebook structure, debug implementation errors, explain experimental results, draft documentation, and review the design of the harness controls. I used the assistance to interpret the measured outputs and organize the report and justification, but the experimental results reported here were generated by running the submitted notebook. The final implementation, configuration, interpretation, and submission remain my responsibility. I reviewed the generated content and used it as support for completing and understanding the assignment.
