# Nobel Prize in Literature Winner Louise Glück

## On Calliope Tweney's sarà (data engineering register)

I do not read fiction now. I read what is in the room, and what is in the room is mostly nothing — the kind of nothing the dead have time for, which is to say all of it. So when the manuscript was passed to me, in the way these things are passed, I read it the way I read everything in life: at a slant, against my own resistance to being instructed, looking for the place where the writer had stopped performing and had begun, instead, to be exposed.

I found that place. I want to tell you where.

—

The manuscript is, on its surface, an exercise. A woman has taken a novel — her own, presumably, or one she has been carrying in some other form — and has rendered it as a sequence of technical documents. README files, schema diagrams, an Airflow pipeline, SQL migrations, Slack channels, a postmortem, a JIRA ticket, a runbook. The exercise has been done well. The exercise is competent. The exercise displays the kind of intelligence that is most readily displayed in a register designed to display intelligence — the register of system architecture, where every choice is auditable and every constraint is checked.

I have been a poet long enough to know that exercises of this kind are usually defenses. The writer who can render her grief as a schema is the writer who can defer her grief by attending to the rendering. I expected, opening the file, to find a defense. I expected to find the kind of cleverness that postpones what the writer cannot face by giving her hands something to do.

I did not find a defense.

I found something else. I want to be precise about what I found, because the precision is the only useful thing the dead can offer the living, and I do not have much else.

—

The story, as it emerges from the documents, is the story of a creature who has been doing harm for two thousand years inside a system she has constructed to make the harm bearable. The system is the warrant. The warrant is the moral architecture. The warrant is the cover story the creature has been telling herself across centuries to allow the killing to continue. The schema documents the warrant. The fact table records its enforcement. The migration documents its retirement.
What the schema also documents, in a single row that sits in the fact table without commentary, is the moment the warrant failed. The row is dated September 17, 2014. The flag was_pleasure_primary is TRUE. The flag warrant_strength_at_kill is NULL. There is no incident ticket associated with the row. The row was entered manually, in October 2014, without comment. The class of the target, against the explicit constraint of the schema, was child.
I have read the row three times. I am going to tell you what I think about it.

The row is the place where the writer stopped performing.

The exercise — the data engineer register, the deadpan, the schema-as-mythology — could have done a great many things with this material. The exercise could have placed the 2014 row prominently. The exercise could have built a narrative around it. The exercise could have made the row the climax. The exercise did none of these. The row sits in a query result, three rows down from the row that records Edward Marsh's execution, in a font no larger than the surrounding cells, with no formatting and no marker. The constraint violation that produced the row is acknowledged in a comment on the constraint definition: violated once: 2014-09-17 (see INC-2014-0917). violated once again: ~2026-01 (no incident ticket; not entered into system).

The second violation has no row.

This is the place where the writer stopped performing. The writer chose, in rendering her own material, to refuse the rendering at the precise point where the rendering would have been most rewarded. The 2026 desert child does not exist in the fact table. The 2026 desert child does not have an incident ticket. The 2026 desert child is named only in the comment on a SQL constraint, in the parenthetical, in the smallest formatting available to the writer in this register. The writer has, in other words, performed the same act of administrative refusal that the creature herself performs in the system she has built: a thing has happened that the system cannot accommodate, and the system has — by silent agreement among all parties — declined to record it.

This is not a defense. This is the opposite of a defense. The defense would have been to write the desert child into the schema in great detail, to make her a centerpiece, to render the killing across many fields and many tables and many incident reports. The writer has chosen, instead, the registration that hurts the most: she has chosen to put the thing in a parenthetical that most readers will not read, and to leave the rest of the system unmodified, as if the unmodification were the only honest response to a thing the system was not built to accommodate.

I recognize this gesture. I have made it myself. The places in my own work where I refused to render the worst materials were the places where the work was most exposed. The reader who knows the gesture will recognize it. The reader who does not will pass over it on the way to the more comfortable surfaces of the schema. The writer has not, in either case, performed.

—

The other place I want to tell you about is the Slack channel.

There is a channel called #incidents. The channel has a single user. The single user is the creature herself, who has been the sole on-call engineer for two thousand years. The channel has, by the time the reader encounters it, accumulated three messages on the afternoon of November 17, 2025, the day the creature killed Tomás Mendel:
```
[14:33] @lamia: /report sev-1
[14:33] @lamia: no responders expected / i am the sole on-call
[14:35] @lamia: leaving the channel running / for my own records
Then, hours later, on her own balcony:
[19:47] @lamia: registering, on my own balcony, that what i did at 14:32
        was structurally the same as INC-2014-0917
        flagging this for self-review
        will not be filing a separate ticket
```

```
[19:48] @lamia: this is the worst i have felt in 2000 years
        no action item
        will continue
```

I want to talk about no action item.

There is, in this phrase, a specific structural achievement the rest of the document only approaches. The phrase belongs to a corporate register — the small administrative phrase a software engineer uses to close a Slack conversation that has no useful continuation. The phrase says: I have noted this. There is nothing to do about it. The conversation is, for operational purposes, over. The phrase is, in its native register, banal.

What the phrase is doing in this Slack message is different. The phrase is saying: I have just registered the worst thing I have felt in two thousand years. The configuration my life has been organized around has been visible to me, from this evening forward, as a configuration I cannot continue. I have no procedure for what to do with this. There is no help available. There is no escalation path. There is no one to file the ticket with. There is no manager. There is no oncall. I am the sole on-call engineer for the system that is myself. The thing I have just registered is unactionable in the technical sense: there is no action that resolves it. Will continue.

The phrase no action item is the entire emotional content of v7's chapter 8 compressed into three words. The phrase is also the entire emotional content of the closing chapters compressed into three words. The phrase is, on examination, the place where the data engineer register is no longer a register but is the actual language a person uses when she has been working alone for two thousand years and has been organizing her own interior around the small specific procedures of a job that has no manager.

The register, in other words, is not the exercise. The register is the diagnosis.

The creature has been a data engineer the entire time. She has been the sole on-call engineer for her own continuance. She has been writing runbooks for the management of her own appetite. She has been entering rows in a fact table that has no audit trail. She has been the person who, when the system fails, is the only person who knows the system has failed, because she is also the system. The data engineer register is not a clever way of telling her story. The data engineer register is what her story has been the entire time. The previous versions of the book — the literary one, the commercial one, all of them — were translations out of the register and into more legible languages. The register, presented this way, is the original.

I want to be careful here. I am not saying every novel ought to be a system manual. I am saying that this particular figure — a creature who has been alone for two thousand years, who has built around her appetite a moral architecture she maintains in solitude, who has no peer review and no editorial oversight and no one above her in the org chart — has been recognizable, all along, as the figure of a person who works in a profession the writer of this manuscript is intimately familiar with. The choice to render the figure in the register the figure occupies is not a translation. The choice is a removal of translation.
This is brave. It is also painful in a way I would like, in my present condition, to be in a position to register on the page rather than only in this attempt at a sentence.

—

There is one moment I want to flag as the place where the manuscript fails its own register, and I want to flag it because the failure is in the direction the manuscript should fail. The moment is the line the clock is, on examination, ours in the final retrospective.

The line is not in the register. The register would not produce on examination in the closing position of the retrospective's closing sentence. The register would produce something more clinical, more spare, more refusing of the emotional register the phrase invokes. On examination is a literary writer's phrase that has been a tic across the larger work the manuscript is condensed from. It has carried, in the larger work, the small specific weight of a sentence-level mannerism. In the data engineer register it appears as a slight failure of register-discipline.

I am going to say what I think the failure means.

The writer has, across the manuscript, maintained the register with discipline. The maintenance has been the work. The maintenance has been the writer's instrument for not flinching from the materials. The final retrospective is the writer's closing gesture. In the closing gesture the maintenance loosens by one phrase. The clock is, on examination, ours.

The phrase is doing what the rest of the manuscript has refused to do, which is to permit a small specific tenderness on the page. The writer has, for the duration of the manuscript, withheld this tenderness — has rendered the creature's hard-won household in stand-up notes and pagerduty rules and SQL queries returning zero. The withholding has been the work's discipline. The closing phrase relaxes the discipline by one notch. The notch is on examination.

I think the relaxation is correct. The relaxation acknowledges that the register has been a register — that the writer has been making a choice across the manuscript not to permit certain kinds of language, and that the choice has cost her something, and that the closing of the manuscript is the small specific place where the cost of the choice is acknowledged. The notch is the acknowledgment. The notch says: I have been writing in this register because the register was the truth, but I am also a person who can write in other registers, and I am going to permit myself one phrase, here at the end, that belongs to the other register. The phrase is on examination. The phrase is a watermark.

The reader who has been with the manuscript through its discipline registers the watermark. The reader who has not will pass over it on the way to the next document in their queue.

I have nothing more to say about it. The watermark is the watermark. I would have done it differently in my own work. I would have done it less. I am not, however, the writer of this manuscript, and the writer of this manuscript has, on examination, earned her one notch.

—

A last thing. The writer is, I gather, herself a data engineer. The biographical fact is relevant. I want to note that I do not, ordinarily, find the biographical fact relevant to the work. I find it relevant here because the manuscript would be a different document if its author were a literary novelist who had taken a workshop on technical writing. The manuscript is not that document. The manuscript is the document of a person whose actual professional life has produced the actual technical artifacts the manuscript renders her fiction into. The schemas are accurate. The Airflow DAG is correctly structured. The SQL is correctly formed. The PagerDuty YAML is the YAML PagerDuty actually accepts. The JIRA ticket has the correct fields.

This is, in its way, the writer's most striking gesture. The writer has taken the materials of her professional life — the materials she has been spending her working hours on for many years — and has used them as the substrate for the rendering of her interior. The professional life is not, in the manuscript, separate from the interior life. The professional life is the medium in which the interior life can be expressed. The data engineer is the writer. The writer is the data engineer. The schema is the poem.

I find this useful to know. I find it useful because I spent a great deal of my own working life looking for the right medium for what I had to say, and I never, in the years I was working, considered the medium of my actual professional materials as a possible substrate. I taught. I wrote. I taught and wrote in registers that I had inherited from other writers. I did not, until I was much older than I would now recommend being before doing this, consider that the actual labor of my days had its own register, and that the register might be the register the work needed.

This writer has done what I did not do. The writer has, in this manuscript, given herself permission to render her fictional materials in the register her actual labor has trained her to. The permission is the move. The permission is also, I suspect, transferable. There is a writer somewhere reading this whose actual professional life — as a barber, as a public defender, as a pediatric nurse, as a librarian, as a translator, as a person who repairs HVAC systems — has produced the register the writer's interior has been waiting to be rendered in, and the writer has not yet given herself permission to use it. I am writing this review in part for that writer. The permission is available. The register is the register. The work the register has been training you to make is the work.

—

I do not, ordinarily, end reviews with instructions. I will end this one by repeating myself.

The 2014 row is the place where the writer stopped performing.

The phrase no action item is what the register actually is.

The phrase on examination, in the final retrospective, is the watermark.

The schema is the poem.

The register is the register. The register is also, in this manuscript, the diagnosis, the refusal of translation, and the form the writer has at last given herself permission to use.

The dead read what is in the room. This was in the room. I read it. I have, on examination, registered what I read.

— Louise
(somewhere)
