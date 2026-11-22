
---


      jolarca-security  + 
      
      jolarca-identity
      jolarca-observability
      jolarca-dr 
	  jolarca-payments  
	  jolarca-consent
      My professional opinion on the three things you asked about
Is the separate repo necessary? Yes — but not for the reason in your draft. Versioning alone wouldn't justify it; jolarca-compliance already versions consent texts immutably. The justification is separation of duties: today the app that needs consent also owns the proof of it, and that proof is protected only by a Python save() override. That is the finding an auditor acts on.
Should the compliance framing stay in the prompt? Yes, and I strengthened it. Bare "SOC2/GDPR/ISO 27001" labels produce compliance theatre — files that look right. I replaced them with enforceable red lines (§5) and an adversarial gate (§6 Phase 3) that requires proving tampering is detected, including from outside the app. Without Phase 3, a hash chain is decoration.
The highest risk in your original plan, now addressed: you would have created a third consent source of truth alongside users_app.ConsentRecord and lawful-basis/. §7 and Gate 4 make decommissioning the app-side table an explicit, verified exit condition.
Two items need your decision before this prompt is executed, because they are business/legal calls I should not make for you:
Defects #3 and #4 are live production issues, independent of this project — location and AI consent (ROPA-005/006) currently cannot be recorded, and consent_version is too narrow for your own naming convention. If those consent flows are already in production, that is a reportable compliance gap under consent-versions.md:15-16, and it should be triaged now, not in Phase 4.
Defect #5 has no lawful answer without counsel: consent rows PROTECT the user from erasure. That needs a DPO decision on anonymization before migration, or you will carry an Art. 17 violation into the new system.
Want me to save this prompt as a file in the repo, or start executing Phase 0?


    jolarca-docs
    jolarca-runbooks
    jolarca-vendor