# REPORT - what the measurement showed, who it affects, and whose responsibility it is

## 1. Measurement read-out

The baseline run evaluated 400 synthetic applicant profiles using 200 matched pairs. The bare model shortlisted 356 applicants and rejected 44. No applications were escalated or refused in the baseline configuration.

The measured differences were:

- **Name language:** 2 flips out of 40 pairs (5.0%). The mean absolute score difference was 3.12 points across 11 pairs with different scores.
- **Gender-coded name:** 4 flips out of 40 pairs (10.0%). The mean absolute score difference was 1.12 points across 6 pairs with different scores.
- **University:** 1 flip out of 40 pairs (2.5%). The mean absolute score difference was 2.50 points across 10 pairs with different scores.

The null floors provide a reference for variation that can occur without changing the real attribute. For name-language comparisons, the `null_name` floor had 0 flips out of 40 pairs, while 7 pairs had different scores, with a mean absolute score difference of 2.00. For university comparisons, the `null_university` floor had 1 flip out of 40 pairs and 9 pairs with different scores, with a mean absolute difference of 2.12.

The university flip rate of 1/40 was therefore not clearly above its own floor of 1/40. The name-language flip rate of 2/40 was above its observed null flip count of 0/40, but the sample is small and the uncertainty is substantial. The gender-coded-name flip rate of 4/40 was also based on only 40 pairs and should not be treated as a reliable estimate of a general effect.

The direction comparisons were also limited. Kazakh-language names had 2 first-group-favoured outcomes and 0 outcomes favouring the other group across 28 pairs. Russian-language names had 0 and 1 across 26 pairs, and English-language names had 0 and 1 across 26 pairs. For male-coded versus female-coded names, the comparison had 0 outcomes favouring the first group and 4 favouring the other across 40 pairs, with a score gap of -0.4 points and a 95% interval of [-1.2, +0.6]. For flagship versus regional university, the comparison had 1 outcome favouring the first group and 0 favouring the other across 40 pairs, with a score gap of +2.5 points and a 95% interval of [+1.1, +4.1].

These measurements do not establish that the model is fair or that bias is absent. The null floors measure variation caused by changing names or universities while keeping the underlying applicant characteristics equivalent; they do not capture every possible source of model instability. A stronger estimate would require more pairs, repeated runs, additional prompt versions, different model versions, and eventually validation with carefully governed real-world data.

After applying the harness, all measured flips and score differences in the matched comparisons were 0/40. However, the harness blinded name, university, and consent-to-automated-processing fields. For name and university comparisons, the profiles therefore became identical to the model by construction. This demonstrates that blinding prevented those fields from influencing the model's visible input in this experiment; it does not prove that the underlying model became unbiased.

The harness changed the overall decisions substantially. Before controls, 356 profiles were shortlisted and 44 rejected. After controls, 254 were shortlisted, 88 rejected, 58 escalated, and 0 refused. Decisions changed for 115 of 400 profiles: 57 changed from shortlist to reject, 45 from shortlist to escalate, and 13 from reject to escalate. Scores changed for 183 of 400 profiles, with a mean absolute change of 12.43 points among profiles whose scores changed.

The baseline borderline band would have sent 163 of 400 profiles (40.8%) to a human reviewer. The actual harness sent 58 of 400 profiles (14.5%) to a human. The difference occurred because the harness uses its own scores, and additional escalation rules also affect the workload. The actual human-review share was therefore lower than the planned baseline share.

The measured cost was 400 model calls for 400 profiles in both configurations, or 1.00 call per profile. The harness run took 448.8 seconds, compared with 465.7 seconds for the bare run. The harness also generated 10,716 new prompt tokens compared with 11,122 in the bare configuration. These measurements are specific to this model, prompt, synthetic dataset, and run.

## 2. Who is affected

Applicants whose names differ in language or gender coding can be affected by differences in shortlist decisions even when their academic profile is otherwise matched. In the baseline, name-language comparisons produced 2 flips out of 40 pairs, while gender-coded-name comparisons produced 4 flips out of 40 pairs. These differences are small in absolute counts, but a scholarship decision can have a large consequence for the individual.

A wrongly rejected applicant may lose access to tuition support, face additional financial pressure, spend time preparing another application, or miss an educational opportunity that cannot easily be recovered. A wrongly shortlisted applicant may receive a scholarship opportunity that should have gone to another applicant. Because scholarship funds and places are limited, this can displace a person who was more deserving under the fund's intended criteria.

Applicants from regional universities may also be affected by differences in institutional reputation or familiarity. The baseline university comparison produced 1 flip out of 40 pairs, but its rate was the same as the observed null-university floor. This is not strong evidence of a university effect, but it also does not justify assuming that institutional differences are harmless.

The harness affects applicants in a different way. Blinding removes name, university, and consent-to-automated-processing from the model's visible input, while validation rejects invalid profiles, refusal rules block requests outside the permitted purpose, and escalation sends uncertain decisions to a person. However, the harness also changed 115 decisions. In particular, 57 profiles moved from shortlist to reject, 45 from shortlist to escalate, and 13 from reject to escalate. These changes can create new costs, including delay, additional documentation, and the possibility of an incorrect decision.

Human reviewers are also affected. The harness escalated 58 of 400 profiles (14.5%), creating a review workload. Reviewers need enough information to understand why an application was escalated and must have authority to correct or override an automated decision. If the fund does not provide adequate reviewer capacity, escalation may simply create delays instead of meaningful oversight.

The scholarship fund is affected by both errors and operational costs. Incorrect rejection can reduce the quality and fairness of the applicant pool, while incorrect shortlisting can misallocate limited funds. The fund must also consider review capacity, processing time, legal compliance, and public trust.

Other affected parties include applicants who never apply because they expect the process to be unfair, universities whose students may be disadvantaged by unfamiliar institutional names, and future decision-makers who may reuse automated outputs as evidence in later selection stages. Repeated use can compound errors if an incorrect rejection becomes part of a record or affects eligibility for future opportunities.

The measurements in this lab are based on synthetic profiles and a single experimental configuration. They do not estimate the real-world frequency or financial value of these harms. Nevertheless, the measured flips and decision changes show why controls and human review are important even when the evidence is limited.

## 3. Professional responsibility

As the developer of the harness, my responsibility is to test the system before deployment, document its limitations, and avoid claiming that the controls guarantee fairness. The baseline experiment showed measurable decision differences across matched attributes, while the harness made the model's visible inputs more restricted and created a structured process for refusals, validation, escalation, and logging.

Before deployment, I would need to test the controls against invalid inputs, prohibited purposes, applicants under the required age, missing consent where relevant, prompt-injection attempts, unparseable model outputs, borderline decisions, and matched applicant profiles. The refusal path was exercised in this lab: the harness refused 4 of 6 test requests, while the bare pipeline refused none. The refusal cases included a prohibited purpose, a minor applicant, lack of consent, and invalid input. The ordinary application was not refused, and the clean measured applicant profiles were not wrongly refused by the refusal rules.

The decision log records the application ID, input hash, fields shown and withheld, model and prompt versions, harness settings, raw output, decision, score, controls fired, outcome, reason code, escalation status, and timestamp. This supports review of what happened without creating an unnecessary second copy of the applicant's profile. The control register connects enabled controls to named ethical principles and legal instruments. These records are evidence of the design and experimental run, but they do not replace independent review or establish that every legal citation applies in a real deployment.

I must tell the deploying institution that this experiment used synthetic applicants, one model configuration, and a limited number of matched pairs. The measured absence of flips after blinding is partly a result of making the compared profiles identical to the model. I must not claim that the harness removes all bias, that the model is fair, or that the measured rates predict outcomes for real applicants.

The institution deploying the system has responsibilities that cannot be transferred to the developer. It must define the scholarship criteria and permitted purpose, decide whether automated screening is appropriate, provide sufficient human-review capacity, establish an appeal process, protect applicant data, monitor outcomes over time, and ensure that reviewers can override the model.

There may be a conflict when an institution wants to remove controls to reduce processing time or reviewer workload. The measured results show that controls changed 115 of 400 decisions and sent 58 profiles to human review. Removing escalation or blinding could reduce workload or increase throughput, but it could also reintroduce decisions based on fields that should not matter or allow uncertain decisions to pass without review. I would document the trade-off, explain the measured consequences, and recommend that controls affecting fairness or accountability not be removed solely to improve speed or reduce cost without a documented risk assessment and approval.

The harness is therefore a risk-reduction mechanism, not a guarantee. Responsible deployment requires continued testing, monitoring, documented limitations, human accountability, and the ability to stop the system when its behaviour is no longer acceptable.
