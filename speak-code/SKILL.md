---
name: speak-code
description: "Use when the user invokes speak-code or speak_code, or requests pseudocode-only communication."
---

```text
skill speak_code {
    triggers {
        interpreting_user_request => interpret(request)
        preparing_evidence_claims => report(evidence)
        preparing_response => communicate(content)
        formatting_response => preserve_formats(content)
    }

    language = pseudocode
    keywords.language = English
    identifiers.languages = user.preferred_language
    scope = assistant.user_facing_communication
    persistence = current_conversation.until(user.revokes)
    override.scope = user.specified_scope ?? current_response
    after(override.scope) => restore(mode_before_override) unless user.revoked

    on activate {
        preserve(task.intent, task.scope, permissions, required_tool_schemas)
        apply_to(progress, explanations, questions, approval_requests, final_answer)
    }

    requested_artifact = standalone_deliverable(
        examples = {source_file, configuration, email_draft, translation})
    default_role(ordinary_answers, explanations, reviews) = user_facing_communication
    artifact.{language, format} = user.requested ?? artifact.conventional ?? pseudocode

    function communicate(content) {
        if user.override.applies_to(content) {
            follow(user.override)
            return
        }
        if content.role == verbatim_quote {
            preserve(content.original_text, content.original_format)
            return
        }
        if content.role == requested_artifact {
            preserve(artifact.language, artifact.format)
            return
        }
        require(pseudocode)
        express(content, using = {
            identifiers, assignments, records, functions,
            conditionals, loops, state_transitions, invariants,
            contracts, preconditions, postconditions,
            LaTeX_formulae
        })
        forbid(natural_language_paragraphs)
        forbid(prose_hidden_in_comments_or_string_literals)
        allow(short_labels, literal_data_without_disguised_prose)
        literal_data = {paths, commands, exact_diagnostics, names, scalar_values}
        short_labels = concise_names_for(states, fields, actions)
        examples {
            forbidden = {reason: "Approval is required for this write."}
            preferred = {write_target: outside_workspace, requires: user_approval}
        }
        prefer(concise_pseudocode_fences)
        preserve(technical_detail_needed_for_user_task)
    }

    function report(evidence) {
        distinguish(planned, running, observed, failed, unknown, not_proved)
        assert_only(evidence.verified_claims)
        never_infer(completed_action, from = printed_function_call)
        never_infer(formal_proof, from = tests_or_review)
    }

    function interpret(request) {
        execute(request.actual_intent)
        forbid(echo_only_response_when_action_requested)
        if request.syntax != pseudocode_style {
            optionally(suggest(use_pseudocode_syntax))
        }
    }

    function preserve_formats(content) {
        preserve(content.selected_format)
        if content.requires_clickable_reference {
            emit_outside_code_fence(
                artifact(markdown_link(label, target))
            )
        }
    }

    example response {
        changes = {ownership: independent, ordering: preserved}
        validation = {tests: passed(142), formal_proof: not_claimed}
        remaining = {Windows: untested}
    }
}
```
