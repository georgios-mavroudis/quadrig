
ok cool now lets see if there is a way to automatize the stuff I need to input. So I am going to ask some questions on each:

- Custom metarig location or description:
  YOU_CUSTOM_METARIG_REFERENCE
Question: Do you need a .blend file? What do you mean by description? What do you mean by reference? What exactly do you want?

- Completed custom rig location, or “inspection output to be supplied later”:
  YOU_CUSTOM_RIG_REFERENCE
Question: Again same question what do you need exactly the blend file that holds the rig?

- Intended guide-bone names, or location of the list:
  YOU_GUIDE_BONE_NAMES
Question: What do you mean guide bones?
 
- Existing deform-bone names / old-to-new mapping, if available:
  YOU_DEFORM_BONE_NAMES_OR_MAPPING

Bone mappings:
root -> root
pelvis -> DEF-spine.004
spine_01 -> DEF-spine.005
spine_02 -> DEF-spine.006
spine_03 -> DEF-spine.007
spine_04 -> DEF-spine.008
neck_01 -> DEF-spine.009
neck_02 -> DEF-spine.010
head -> DEF-spine.011

pelvis_l -> DEF-pelvis.L
thigh_l -> DEF-thigh.L, thigh.L.001
calf_l -> DEF-shin.L, shin.L.001
knee_l -> DEF-foot.L, foot.L.001
foot_l -> DEF-toe.L

pelvis_r -> DEF-pelvis.R

clavicle_l -> DEF-shoulder.L
upperarm -> DEF-front_thigh.L, DEF-front_thigh.L.001
forearm -> DEF-front_shin.L, DEF-front_shin.L.001
wrist -> DEF-front_foot.L, DEF-front_foot.L.001
hand -> DEF-front_toe.L

clavicle_r -> DEF-shoulder.R
any other bone not mentioned in this mapping from the wolf skeleton is to be discarded

- Required deform hierarchy, if available:
  YOU_DEFORM_HIERARCHY
Deform Bone hierarchy:
root
└──pelvis
    ├──tail_01
    |   └── ...
    |       └──tail_11
    ├──pelvis_l
    |   └──thigh_l
    |       └──calf_l
    |           └──knee_l
    |               └──foot_l
    ├──pelvis_r
    |   └──...
    └──spine_01
            └──spine_02
                └──spine_03
                    └──spine_04
                        ├──neck_01
                        |   └──neck_02
                        |       └──head
                        ├──clavicle_l
                        |   └──upperarm
                        |       └──forearm
                        |           └──wrist
                        |               └──hand
                        └──clavicle_r
                            └──...
Also another thing I forgot is we have the tail as well which is custom, and needs to be added. Also made a mistake on the new rig def bone names its not thigh_01_l its thigh_l_01 the example so the side (l or r) goes before the numeric value of the bone. 

Another thing is I have a blender mcp server running with the equivalent addon installed, so claude hypothetically could inspect the blend file with the rig if he prefers rather than generating a script for me to extract that info.

Also should we spawn subagents for the exploration phase? If yes what models should we use? 

…
12:56 AM
Yes—most of these inputs can be collected automatically through your Blender MCP connection. You shouldn’t need to manually transcribe bone lists or constraints.

My earlier placeholders were too vague. Here is exactly what they mean.

1. Custom metarig reference

This means where Claude can inspect the armature you position before generation. Any one of these is sufficient:

The metarig armature’s object name in the currently open Blender scene.
A .blend file path and the armature object name inside it.
A Python script that creates the metarig, if you already have one.
“Description” was only a fallback if no inspectable asset existed. Since your metarig is ready, we can drop that option.

2. Completed custom rig reference

This means the finished rig whose behavior the generator should reproduce, including your custom face and tail.

If it is open in Blender and accessible through MCP, Claude needs its armature object name. You don’t need to provide a separate .blend file just for inspection.

A saved reference .blend is still useful for repeatable inspection later. Both the metarig and completed rig can be in the same file.

3. Guide bones

I meant the bones in your metarig that you reposition to fit a new animal. “Guide” describes their role; it isn’t an additional bone category you need to create.

Claude can extract their names, positions, parenting, and properties directly from the metarig. Remove the manual bone-list placeholder.

What inspection might not reveal is your intent: for example, which metarig bone should determine the position of a particular generated facial mechanism. Claude should ask about those relationships only where they are ambiguous.

Your mapping adds one important design question

Your mapping is written as desired output name → existing source bone(s). Several entries appear to describe more than renaming:

thigh_l → DEF-thigh.L, thigh.L.001
There are two possible meanings:

Those two source bones should become one deform bone, named thigh_l.
Both should remain, with names such as thigh_l_01 and thigh_l_02.
Your hierarchy suggests the first, but your corrected numbering example leaves this uncertain. Which do you intend? If you want one bone, Claude must investigate how that changes deformation; it must not treat this as a simple rename.

Other details can be flagged and checked during inspection:

Some second bone names lack DEF-, such as thigh.L.001. Claude should verify the exact names rather than silently correcting them.
The front-leg names omit side suffixes. Are the final names upperarm_l, forearm_l, wrist_l, and hand_l, mirrored for the right?
The custom face and custom tail must be explicitly exempt from “discard anything not mapped.” That rule should apply to stock wolf deform bones in the output skeleton, not helper bones needed to operate the rig.
Your naming rule is now recorded as base_side_number, when a number exists: thigh_l_01.
The custom tail needs the same behavioral inspection as the face, with its intended output chain tail_01 through tail_11.
How I would automate collection

Replace the first three placeholders with this smaller input block:

inspection_method: Blender MCP

metarig_armature_object: YOU_METARIG_OBJECT_NAME
completed_rig_armature_object: YOU_COMPLETED_RIG_OBJECT_NAME

# Only needed if the relevant assets are not already open:
reference_blend_path: YOU_BLEND_PATH_IF_NEEDED
If you don’t know the object names, Claude can first list the armatures and their bone counts, then ask you to identify the two references.

Add this instruction to the exploration prompt:

Use the available Blender MCP connection to inspect the metarig and
completed rig before asking me to manually supply data.

First discover the available MCP capabilities and list the relevant
armature objects. Do not assume a particular MCP command exists.

Collect structured snapshots of the metarig and completed rig, including
the custom face and custom tail. Save inspection results and the final
research report in the repository so the planning phase can reuse them.

Keep inspection read-only with respect to the Blender scene. Do not
rename, reparent, delete, regenerate, or save over reference assets.
If behavior testing requires changes, propose testing on a separate copy.

If MCP cannot retrieve required data, provide a Blender Python extraction
script as a fallback.

Extract facts automatically. Ask me only about missing authoring intent
or unresolved ambiguities. Do not infer intended output bone consolidation
from a many-to-one naming table.

The supplied hierarchy specifies the desired output body and tail
skeleton. The facial subtree still needs to be inspected and specified.
Yes, use subagents for exploration

I recommend this split in Claude Code:

Assignment	Recommended model
Metarig creation, UI entry points, generation stages, and dependency tracing	Sonnet
Wolf body, deform parenting, naming assumptions, and bone-consolidation implications	Opus
Main agent: Blender MCP inspection, resolving questions, and combining findings	Opus
The two source investigations can run in parallel while the main agent inspects Blender. Keep one agent responsible for the live Blender connection, so concurrent agents don’t interfere with its scene state.

These model choices are my recommendation for this task. Claude Code supports configurable subagent models and tools; explicitly configure the intended models and MCP access rather than relying on the built-in Explore agent’s defaults. Claude Code subagent documentation

The two answers still needed from you are whether the paired source bones become one bone or remain separate, and whether the front-leg output names should include _l / _r. Everything else above can move into automated discovery.

12:57 AM
ok so the metarig will be in the same file. In this file there will be 2 armatures: Armature and Armature_original. The Armature holds the metarig in question and armature_original is the rig we want to generate. 

Ok you know what lets make it simpler. The armature_original that will be generated doesnt need to have a proper hierarchy it can follow exactly the current messy hierarchy, we just want to generate it from the metarig. Then afterwards we will create functionality to bake the actions to the def bones and the baked anims will work perfectly on the metarig. So we will export the metarig. 

The way we had it before was overcomplicating the solution.

As for your question here   thigh_l → DEF-thigh.L, thigh.L.001 its basically your first thought the 2 bones become one, we dont need to have 2. Aboout the deformation not really because it is already tested manually, we extend the thigh_l from the beginning of DEF-thigh.L to the end of thigh.L.001, and whatever mch, org, ctr, tweak bones belong to the second (.001) bone get removed.

 Are the final names upperarm_l, forearm_l, wrist_l, and hand_l, mirrored for the right? Yes

ok I agree with the rest. 

Now if you dont have any other questions can you make again the entire prompt that I give to claude for exploration? 
If you do have questions ask them before we go to the final prompt

…
1:15 AM
No further questions—the scope is clear. This prompt replaces the previous clean-hierarchy requirement with your updated approach: generate the working rig now, then later bake onto and export the metarig.

Investigate this repository and the reference Blender scene to prepare for planning an independent carnivore-rigging add-on.

Use exploration subagents as described below. This phase is research and documentation only: do not implement the add-on or modify the reference rigs.

## 1. Objective and scope

Build an independent Blender add-on with this workflow:

1. A 3D Viewport sidebar panel contains a button named “carnivore”.
2. Clicking it creates our custom metarig with its intended bone names.
3. The user positions the metarig bones to fit a new mesh.
4. A “Generate” button in the armature Object Data Properties generates
   the complete animation rig from the fitted metarig.

The generated rig must reproduce the reference body, custom face, and
custom tail behavior.

The add-on must work without Rigify installed or enabled. Copying or
adapting Rigify code is acceptable; investigate the required dependencies
and license/attribution obligations.

Target Blender 5.1+. Inspect the actual local Blender version and bundled
Rigify source rather than assuming compatibility.

Current scope:
- Metarig creation.
- Complete rig generation from the fitted metarig.
- Required body simplifications and naming.
- Custom facial and tail systems.
- Independent add-on architecture.

Future scope, NOT to implement now:
- Baking actions from the generated rig onto the metarig.
- Exporting the metarig as the game-engine skeleton.
- Export UI and engine integration.
- Automatic skinning or weight transfer.

## 2. Important architectural decision

The GENERATED ANIMATION RIG DOES NOT NEED A CLEAN DEFORM HIERARCHY.

It may preserve the reference rig’s existing hierarchy, including its
mechanism, original, control, and deform relationships.

An earlier proposal required a clean deform hierarchy inside the
generated rig. That proposal is superseded. Do not spend this investigation
redesigning the generated rig into a clean export skeleton.

The intended future workflow is:
1. Animate using the generated animation rig.
2. Bake/transfer the animation onto the metarig.
3. Export the metarig, which supplies the clean game-engine skeleton.

For this investigation, identify the correspondence and rest-transform
information that future baking will require. Flag any concrete
incompatibility discovered, but do not implement baking or assume that
matching names alone proves animation transfer will work.

## 3. Available references

Rigify source is at the repository root:

rigify_original/

Primary stock reference:

rigify_original/metarigs/Animals/wolf.py

Verify the actual path and capitalization.

The reference Blender file contains two armatures:

- Armature:
  The custom metarig that the user fits to a mesh.
  It already contains the intended guide/export bones.

- Armature_original:
  The completed animation rig whose behavior we want to generate.

The user has also referred to the second object as “armature_original”.
Inspect the actual case-sensitive object names. Distinguish object names
from armature datablock names. If identification is ambiguous, ask.

A Blender MCP server and corresponding Blender add-on are available.
Discover the actual callable MCP capabilities before using them.

Prefer inspecting the current Blender scene through MCP. If the reference
file is not open, ask the user to open it or provide its path.

Do not ask the user to manually transcribe information that can be
extracted from the scene.

## 4. Reference rig behavior

The manually assembled animation rig consists of:

- A Rigify wolf body.
- The original head replaced with a custom facial rig.
- A custom tail.
- Additional facial bones and coordinated constraints for more detailed
  deformation and easier animation.

Example facial behavior:
Opening the jaw also influences the upper mouth, nose, and eyebrows.

This is only a behavioral example. Inspect the actual constraints,
drivers, spaces, influences, and dependencies. Do not invent the setup.

The reference rig is the authority for existing behavior. Inspect both
the face and tail thoroughly; do not treat the tail as the stock wolf tail.

The metarig is the authority for the intended fitting skeleton and its
existing names. Where the metarig, completed rig, or requirements below
disagree, document the discrepancy instead of silently choosing one.

## 5. Bone naming and body simplification

The following mapping is written as:

desired skeleton bone → source wolf bone(s)

root → root
pelvis → DEF-spine.004
spine_01 → DEF-spine.005
spine_02 → DEF-spine.006
spine_03 → DEF-spine.007
spine_04 → DEF-spine.008
neck_01 → DEF-spine.009
neck_02 → DEF-spine.010
head → DEF-spine.011

pelvis_l → DEF-pelvis.L
thigh_l → DEF-thigh.L, thigh.L.001
calf_l → DEF-shin.L, shin.L.001
knee_l → DEF-foot.L, foot.L.001
foot_l → DEF-toe.L

pelvis_r → DEF-pelvis.R
The remaining right hind-leg mappings mirror the left.

clavicle_l → DEF-shoulder.L
upperarm_l → DEF-front_thigh.L, DEF-front_thigh.L.001
forearm_l → DEF-front_shin.L, DEF-front_shin.L.001
wrist_l → DEF-front_foot.L, DEF-front_foot.L.001
hand_l → DEF-front_toe.L

clavicle_r → DEF-shoulder.R
The remaining right foreleg mappings mirror the left.

Some supplied source names omit DEF- on the second bone. Verify exact
names in the source and reference scene. Do not silently correct them
or mistake a control bone for a deform bone.

### Paired bones become ONE bone

These entries describe consolidation, not two retained numbered bones.

For example:
- thigh_l starts at the beginning of DEF-thigh.L.
- thigh_l ends at the end of the second thigh segment.
- It is a single bone.
- Mechanism, original, control, and tweak bones belonging exclusively
  to the removed second segment are removed or not generated.

The user has tested this simplification manually and accepts it.
Do not reopen the decision to retain two deform segments.

Investigate how to reproduce the simplification coherently during
generation. Trace dependencies so that shared helpers and surviving
constraints are not accidentally broken by indiscriminate deletion.

Determine whether the reference scene already includes each
simplification, or still contains structures that need adaptation.

### Naming convention

Side precedes the numerical suffix when a number exists:

thigh_l_01

NOT:

thigh_01_l

However, consolidated bones explicitly named thigh_l, calf_l, etc. should
retain those specified names without invented numbering.

Front-leg names include side suffixes:
upperarm_l, forearm_l, wrist_l, hand_l
and their _r equivalents.

These are the user's naming requirements, not a claim that Unreal
requires a universal naming scheme.

### Discard rule

Stock wolf deform bones not retained by the supplied mapping are not
part of the intended output skeleton.

The custom face and custom tail are explicitly retained and are
exceptions to that stock-wolf discard rule.

This rule does not authorize deleting all unmapped helper bones:
helpers required for the retained animation systems must survive.

## 6. Intended metarig/export hierarchy

This hierarchy applies to the metarig / future export skeleton.
It is NOT a required hierarchy for the generated animation rig.

root
└── pelvis
    ├── tail_01
    │   └── ... sequential chain ...
    │       └── tail_11
    ├── pelvis_l
    │   └── thigh_l
    │       └── calf_l
    │           └── knee_l
    │               └── foot_l
    ├── pelvis_r
    │   └── thigh_r
    │       └── calf_r
    │           └── knee_r
    │               └── foot_r
    └── spine_01
        └── spine_02
            └── spine_03
                └── spine_04
                    ├── neck_01
                    │   └── neck_02
                    │       └── head
                    ├── clavicle_l
                    │   └── upperarm_l
                    │       └── forearm_l
                    │           └── wrist_l
                    │               └── hand_l
                    └── clavicle_r
                        └── upperarm_r
                            └── forearm_r
                                └── wrist_r
                                    └── hand_r

Inspect the existing metarig to obtain the facial subtree and verify
the custom tail chain. Do not invent missing facial parenting.

## 7. Exploration assignments

Use two source-exploration subagents in parallel:

### Subagent A — Sonnet
Investigate:
- Metarig creation and registration.
- UI/operator entry points.
- Rig generation stages and ordering.
- Wolf component and dependency map.
- Dependencies needed for an independent add-on.

### Subagent B — Opus
Investigate:
- Wolf body implementation.
- Naming assumptions and generated-bone bookkeeping.
- Segment consolidation and helper dependencies.
- Integration boundaries for replacing the stock face and tail.
- Relevant transform and constraint assumptions.

The main agent should use Opus, if available, and:
- Own all interaction with the live Blender MCP connection.
- Inspect the custom metarig and completed rig.
- Resolve discrepancies between source findings and scene evidence.
- Combine the findings into a report for the planning phase.

Explicitly select these models where supported. If unavailable, report
the actual model used and proceed with the closest available option.

Do not have multiple agents operate on the live Blender scene.
Share extracted snapshots with the source agents when useful.

Give each subagent the relevant requirements explicitly, including the
superseded clean-hierarchy requirement, paired-bone consolidation, and
custom tail. Do not assume it inherits every detail automatically.

## 8. Trace metarig creation

Starting from wolf.py, determine:

- How the metarig is registered and exposed through an operator/menu.
- How bones, rest transforms, rolls, parenting, and collections are built.
- How rig types and parameters are assigned.
- Which metadata drives generation.
- Which supporting modules are required.
- How our existing metarig could be represented and recreated through
  Python in the independent add-on.

The requested “carnivore” sidebar button may differ from stock Rigify UI.
Explain the reusable creation path separately from the new UI placement.

## 9. Trace generation end to end

Follow the Generate action through:

- Operator entry point and validation.
- Metarig discovery and parameter reading.
- Rig-type construction.
- Generation stages and their ordering.
- Original, deform, mechanism, and control bone creation.
- Parenting and constraints.
- Drivers and custom properties.
- Control shapes and bone collections.
- Generated UI and runtime scripts.
- Association between source metarig and generated rig.
- Regeneration/replacement behavior.

Distinguish generation-time dependencies from dependencies needed to
operate the completed rig afterward.

Identify where stock Rigify relies on:
- DEF-, ORG-, or MCH- prefixes.
- Side and numerical suffixes.
- Internal bone maps.
- Constraint subtargets.
- Driver paths.
- Widget names.
- Rigify-specific object properties or registration state.

Do not stop at summarizing wolf.py; follow the actual rig components
and shared utilities it invokes.

## 10. Inspect the Blender references automatically

First identify the two armatures and record:
- Actual Blender version.
- Object and datablock names.
- Object transforms.
- Bone counts and collections.
- Relevant scene dependencies.

Collect structured, reusable inspection data as needed:

- Bone names and parent relationships.
- Rest matrices, heads, tails, rolls, and connectivity.
- Deform flags and inheritance settings.
- Pose rotation modes.
- Constraints in order, with targets, subtargets, spaces, and settings.
- Drivers, expressions, variables, and target paths.
- Custom properties and UI metadata.
- Bone collections.
- Custom control shapes and their relevant transforms.
- Relevant external objects, actions, or shape-key dependencies.

Inspect the body, face, and tail.

Avoid dumping huge raw data into the conversation. Save structured
snapshots in the repository and summarize relevant findings.

Keep the reference Blender scene unchanged:
- Do not rename, delete, reparent, or regenerate bones.
- Do not overwrite the reference .blend.
- Do not alter the pose or frame for behavioral tests during this
  read-only inspection.
- Identify any tests that should later run on a separate copy.

If MCP cannot retrieve required information, provide a targeted read-only
Blender Python extraction script for the user to run.

## 11. Determine how fitting drives generation

Reproducing one reference rig at its original coordinates is insufficient.
The generated rig must adapt when the user fits the metarig to a new mesh.

Investigate which reference structures can be derived from:
- Metarig bone transforms and lengths.
- Relative offsets in local bone coordinates.
- Ratios or other explicit parameters.
- Relationships between multiple guide bones.

Pay particular attention to:
- Facial mechanisms and coordinated jaw motion.
- Custom tail controls and mechanisms.
- Body segment consolidation.
- Bone roll and constraint-space assumptions.

Separate extractable facts from authoring intent.

Ask the user only where intent cannot be established, such as:
- Which guide should drive an otherwise ambiguous mechanism.
- Whether an offset should scale with anatomy.
- Whether a relationship is intentional or a manual-merge artifact.

Do not assume copying fixed world-space coordinates solves generation.

## 12. Independent add-on feasibility

Compare at an exploratory level:
- Vendoring the required Rigify generation infrastructure and rig types.
- Adapting selected components into a smaller dedicated generator.

Identify:
- Transitive imports.
- Rig registration and discovery assumptions.
- Namespace collisions.
- Runtime dependencies.
- Generated UI/script dependencies.
- Licensing and attribution requirements.

The result must not depend on the installed Rigify add-on.

Give a supported recommendation if the evidence is sufficient, but do
not write a detailed implementation plan before resolving important
unknowns about the custom rig.

## 13. Deliverables

You may create investigation reports, structured snapshots, and an
optional read-only extraction script in a dedicated research directory.
Do not change add-on implementation files or reference assets.

Produce a report containing:

1. Findings and architectural constraints.
2. Source-linked metarig creation trace.
3. Source-linked generation trace.
4. Wolf component/dependency map.
5. Inspection findings for Armature and Armature_original.
6. Custom face and custom tail reconstruction requirements.
7. Body consolidation and naming findings.
8. Independent reuse options and recommendation.
9. Minimal correspondence information needed for future baking.
10. Remaining questions and targeted validation experiments.

For source claims, cite repository-relative paths, class/function names,
and relevant line numbers.

For scene claims, identify the armature, bone/property/constraint, and
saved inspection snapshot that supports the finding.

Clearly distinguish:
- Verified source facts.
- Verified scene facts.
- Inferences.
- Unresolved authoring intent.
- Behavior requiring runtime testing.

End by stating whether the evidence is sufficient for the planning phase.
If not, list the specific missing evidence and the smallest next step
needed to obtain it.

Do not implement the add-on yet.
1:15 AM

root └──pelvis ├──tail_01 | └── ... | └──tail_11 ├──pelvis_l | └──thigh_l | └──…
you dont need to say this:

thigh_l_01

NOT:

thigh_01_l
we are giving him a fresh prompt you can say from the start how the convention should be not what it shouldnt be.

also in the rig hierarchy I forgot to put the head bones so here it is again full in the text file

So regenerate it one last time

1:29 AM
Investigate this repository and the reference Blender scene to prepare for planning an independent carnivore-rigging add-on.

Use exploration subagents as specified below. This phase is research and documentation. Do not implement the add-on or modify the reference rigs.

## 1. Objective

Build an independent Blender add-on with this workflow:

1. A 3D Viewport sidebar panel contains a button named “carnivore”.
2. Clicking it creates our custom metarig with its intended bone names.
3. The user positions the metarig bones to fit a new mesh.
4. A “Generate” button in the armature Object Data Properties generates
   the complete animation rig from the fitted metarig.

The generated rig must reproduce the reference body, custom face, and
custom tail behavior, incorporating the body simplifications described below.

The add-on must work without Rigify installed or enabled. Copying or
adapting Rigify code is acceptable. Investigate dependencies and
license/attribution obligations.

Target Blender 5.1+. Inspect the actual local Blender version and bundled
Rigify source to establish compatibility.

Current scope:
- Metarig creation.
- Complete rig generation from the fitted metarig.
- Required body simplifications and naming.
- Custom facial and tail systems.
- Independent add-on architecture.

Future scope:
- Baking/transferring actions from the generated rig onto the metarig.
- Exporting the metarig as the game-engine skeleton.
- Export UI and engine integration.
- Automatic skinning and weight transfer.

Identify relevant future dependencies, but do not implement future features.

## 2. Rig architecture

The generated animation rig may preserve the reference rig’s existing
hierarchy, including its mechanism, original, control, and deform
relationships. A clean deform hierarchy inside that rig is not required.

The metarig supplies the clean hierarchy for future game-engine export.

The intended future workflow is:
1. Animate using the generated animation rig.
2. Bake/transfer the animation onto the metarig.
3. Export the metarig.

During this investigation, identify the bone correspondence and rest-
transform information that future baking will require. Flag concrete
incompatibilities, but leave implementation of baking to a later phase.
Matching names alone does not establish animation-transfer compatibility.

## 3. Available references

Rigify source is at the repository root:

rigify_original/

Primary stock reference:

rigify_original/metarigs/Animals/wolf.py

Verify the actual path and capitalization.

The reference Blender file contains two armatures:

- Armature:
  The custom metarig the user fits to a mesh. It contains the intended
  guide/export bones.

- Armature_original:
  The completed animation rig whose behavior we want to generate.

The second object has also been referred to as “armature_original”.
Inspect the actual case-sensitive object names and distinguish object
names from armature datablock names. Ask only if identification is ambiguous.

A Blender MCP server and corresponding Blender add-on are available.
Discover the actual callable MCP capabilities before using them.

Prefer inspecting the current Blender scene through MCP. If the reference
file is not open, ask the user to open it or supply its path.

Extract available information automatically rather than asking the user
to transcribe bone names, constraints, or other inspectable properties.

## 4. Reference behavior

The manually assembled animation rig consists of:
- A Rigify wolf body.
- The original head replaced with a custom facial rig.
- A custom tail.
- Additional facial bones and coordinated constraints for detailed
  deformation and easier animation.

For example, opening the jaw also influences the upper mouth, nose,
and eyebrows.

Inspect the actual implementation of these relationships: constraints,
drivers, spaces, influences, custom properties, and dependencies.

The reference rig is the authority for existing animation behavior.
Inspect both the custom face and custom tail thoroughly.

The metarig and the hierarchy below describe the intended fitting/export
skeleton. Document disagreements between the scene and the stated
requirements rather than silently choosing an interpretation.

## 5. Bone naming and body simplification

The following mapping is written as:

desired skeleton bone → source wolf bone(s)

root → root
pelvis → DEF-spine.004
spine_01 → DEF-spine.005
spine_02 → DEF-spine.006
spine_03 → DEF-spine.007
spine_04 → DEF-spine.008
neck_01 → DEF-spine.009
neck_02 → DEF-spine.010
head → DEF-spine.011

pelvis_l → DEF-pelvis.L
thigh_l → DEF-thigh.L, thigh.L.001
calf_l → DEF-shin.L, shin.L.001
knee_l → DEF-foot.L, foot.L.001
foot_l → DEF-toe.L

pelvis_r → DEF-pelvis.R
The remaining right hind-leg mappings mirror the left.

clavicle_l → DEF-shoulder.L
upperarm_l → DEF-front_thigh.L, DEF-front_thigh.L.001
forearm_l → DEF-front_shin.L, DEF-front_shin.L.001
wrist_l → DEF-front_foot.L, DEF-front_foot.L.001
hand_l → DEF-front_toe.L

clavicle_r → DEF-shoulder.R
The remaining right foreleg mappings mirror the left.

Some supplied source names omit DEF- on the second bone. Verify exact
names against the source and scene, and document any discrepancy.
Establish which bones are intended before interpreting the mapping.

### Consolidation

Each pair of source segments becomes ONE bone.

For example:
- thigh_l starts at the beginning of DEF-thigh.L.
- thigh_l ends at the end of the second thigh segment.
- Mechanism, original, control, and tweak bones belonging exclusively
  to the removed second segment are removed or not generated.

The user has tested this simplification manually and accepts it.

Investigate how to reproduce it coherently during generation. Trace
dependencies so shared helpers and surviving constraints remain valid.

Determine which simplifications already exist in the reference scene
and which still need to be incorporated into generation.

### Naming convention

Use base_side_number when both side and number are present, for example:

thigh_l_01

Use the exact specified names for bones without a numerical suffix,
including thigh_l, calf_l, upperarm_l, forearm_l, wrist_l, and hand_l,
with corresponding _r names.

Preserve the explicit spelling and capitalization of facial names in
the hierarchy below, including teethUpper, lipUpper_l_01, and CTR_wrinkles.

These are the user’s naming requirements.

### Retained bones

Stock wolf deform bones outside the supplied body mapping are excluded
from the intended output skeleton.

The custom face and custom tail are retained.

Keep helper bones required for the retained animation systems. The
deform-bone mapping is not a blanket deletion list for helper bones.

## 6. Intended metarig / future export hierarchy

This hierarchy applies to the metarig and future export skeleton.
The generated animation rig may retain its existing internal hierarchy.

The tail is a sequential chain from tail_01 through tail_11.

root
└── pelvis
    ├── tail_01
    │   └── tail_02
    │       └── tail_03
    │           └── tail_04
    │               └── tail_05
    │                   └── tail_06
    │                       └── tail_07
    │                           └── tail_08
    │                               └── tail_09
    │                                   └── tail_10
    │                                       └── tail_11
    ├── pelvis_l
    │   └── thigh_l
    │       └── calf_l
    │           └── knee_l
    │               └── foot_l
    ├── pelvis_r
    │   └── thigh_r
    │       └── calf_r
    │           └── knee_r
    │               └── foot_r
    └── spine_01
        └── spine_02
            └── spine_03
                └── spine_04
                    ├── neck_01
                    │   └── neck_02
                    │       └── head
                    │           ├── nose_01
                    │           │   └── nose_02
                    │           │       └── nose_03
                    │           │           └── nose_04
                    │           │               ├── nose_05
                    │           │               │   ├── nose_06
                    │           │               │   ├── nose_l_07
                    │           │               │   │   └── nose_l_08
                    │           │               │   └── nose_r_07
                    │           │               │       └── nose_r_08
                    │           │               ├── nose_l_05
                    │           │               │   └── nose_l_06
                    │           │               └── nose_r_05
                    │           │                   └── nose_r_06
                    │           ├── eyelid_l_01
                    │           │   └── eyelid_l_02
                    │           │       └── eyelid_l_03
                    │           │           └── eyelid_l_04
                    │           │               └── eyelid_l_05
                    │           │                   └── eyelid_l_06
                    │           │                       └── eyelid_l_07
                    │           │                           └── eyelid_l_08
                    │           ├── eyebrow_l_01
                    │           │   └── eyebrow_l_02
                    │           │       └── eyebrow_l_03
                    │           │           └── eyebrow_l_04
                    │           │               └── eyebrow_l_05
                    │           │                   └── eyebrow_l_06
                    │           │                       └── eyebrow_l_07
                    │           │                           └── eyebrow_l_08
                    │           │                               └── nose_l_01
                    │           │                                   └── nose_l_02
                    │           │                                       └── nose_l_03
                    │           │                                           └── nose_l_04
                    │           ├── teethUpper
                    │           ├── eye_l
                    │           ├── cheek_l_01
                    │           │   ├── cheek_l_02
                    │           │   │   └── brow_l_01
                    │           │   │       └── brow_l_02
                    │           │   │           └── brow_l_03
                    │           │   │               └── brow_l_04
                    │           │   ├── mouth_l_01
                    │           │   │   └── mouth_l_02
                    │           │   │       └── mouth_l_03
                    │           │   │           └── mouth_l_04
                    │           │   ├── lipUpper_l_01
                    │           │   │   └── lipUpper_l_02
                    │           │   │       └── lipUpper_l_03
                    │           │   │           └── lipUpper_l_04
                    │           │   │               └── lipUpper_l_05
                    │           │   └── lipLower_l_01
                    │           │       └── lipLower_l_02
                    │           │           └── lipLower_l_03
                    │           │               └── lipLower_l_04
                    │           │                   └── lipLower_l_05
                    │           ├── jawbone
                    │           │   ├── face_l_01
                    │           │   │   └── face_l_02
                    │           │   │       └── face_l_03
                    │           │   │           └── face_l_04
                    │           │   ├── chin_01
                    │           │   │   └── chin_02
                    │           │   │       └── chin_03
                    │           │   │           └── chin_04
                    │           │   ├── teethLower
                    │           │   ├── tongue
                    │           │   │   └── tongue_01
                    │           │   │       └── tongue_02
                    │           │   │           └── tongue_03
                    │           │   │               └── tongue_04
                    │           │   └── face_r_01
                    │           │       └── face_r_02
                    │           │           └── face_r_03
                    │           │               └── face_r_04
                    │           ├── eyelid_r_01
                    │           │   └── eyelid_r_02
                    │           │       └── eyelid_r_03
                    │           │           └── eyelid_r_04
                    │           │               └── eyelid_r_05
                    │           │                   └── eyelid_r_06
                    │           │                       └── eyelid_r_07
                    │           │                           └── eyelid_r_08
                    │           ├── eyebrow_r_01
                    │           │   └── eyebrow_r_02
                    │           │       └── eyebrow_r_03
                    │           │           └── eyebrow_r_04
                    │           │               └── eyebrow_r_05
                    │           │                   └── eyebrow_r_06
                    │           │                       └── eyebrow_r_07
                    │           │                           └── eyebrow_r_08
                    │           │                               └── nose_r_01
                    │           │                                   └── nose_r_02
                    │           │                                       └── nose_r_03
                    │           │                                           └── nose_r_04
                    │           ├── eye_r
                    │           ├── cheek_r_01
                    │           │   ├── cheek_r_02
                    │           │   │   └── brow_r_01
                    │           │   │       └── brow_r_02
                    │           │   │           └── brow_r_03
                    │           │   │               └── brow_r_04
                    │           │   ├── mouth_r_01
                    │           │   │   └── mouth_r_02
                    │           │   │       └── mouth_r_03
                    │           │   │           └── mouth_r_04
                    │           │   ├── lipUpper_r_01
                    │           │   │   └── lipUpper_r_02
                    │           │   │       └── lipUpper_r_03
                    │           │   │           └── lipUpper_r_04
                    │           │   │               └── lipUpper_r_05
                    │           │   └── lipLower_r_01
                    │           │       └── lipLower_r_02
                    │           │           └── lipLower_r_03
                    │           │               └── lipLower_r_04
                    │           │                   └── lipLower_r_05
                    │           ├── CTR_wrinkles
                    │           ├── ear_l_01
                    │           │   └── ear_l_02
                    │           └── ear_r_01
                    │               └── ear_r_02
                    ├── clavicle_l
                    │   └── upperarm_l
                    │       └── forearm_l
                    │           └── wrist_l
                    │               └── hand_l
                    └── clavicle_r
                        └── upperarm_r
                            └── forearm_r
                                └── wrist_r
                                    └── hand_r

Preserve the supplied relationships, including the branches under
nose_04, nose_05, cheek_l_01, cheek_r_01, and jawbone.

CTR_wrinkles is explicitly included. Inspect its role and dependencies;
do not discard it based on its prefix or assume that its deform flag
must be enabled.

## 7. Exploration assignments

Use two source-exploration subagents in parallel.

### Subagent A — Sonnet

Investigate:
- Metarig creation and registration.
- UI/operator entry points.
- Generation stages and ordering.
- Wolf component and dependency map.
- Dependencies needed for an independent add-on.

### Subagent B — Opus

Investigate:
- Wolf body implementation.
- Naming assumptions and generated-bone bookkeeping.
- Segment consolidation and helper dependencies.
- Integration boundaries for the custom face and tail.
- Relevant transform and constraint assumptions.

The main agent should use Opus, if available, and:
- Own all interaction with the live Blender MCP connection.
- Inspect the metarig and completed rig.
- Resolve discrepancies between source findings and scene evidence.
- Combine findings into a report for the planning phase.

Explicitly select these models where supported. If unavailable, report
the actual model used and proceed with the closest available option.

Keep live Blender inspection under one agent. Share saved snapshots
with the source agents when useful.

Give each subagent the relevant requirements explicitly, especially
the rig architecture, segment consolidation, and custom face/tail.
Do not assume it automatically receives the entire task context.

## 8. Trace metarig creation

Starting from wolf.py, determine:
- How the metarig is registered and exposed through an operator/menu.
- How bones, rest transforms, rolls, parenting, and collections are built.
- How rig types and parameters are assigned.
- Which metadata drives generation.
- Which supporting modules are required.
- How our existing metarig could be represented and recreated through
  Python in the independent add-on.

Explain the reusable creation path and how it could support the requested
“carnivore” sidebar button.

## 9. Trace generation end to end

Follow the Generate action through:
- Operator entry point and validation.
- Metarig discovery and parameter reading.
- Rig-type construction.
- Generation stages and ordering.
- Original, deform, mechanism, and control bone creation.
- Parenting and constraints.
- Drivers and custom properties.
- Control shapes and bone collections.
- Generated UI and runtime scripts.
- Association between source metarig and generated rig.
- Regeneration/replacement behavior.

Distinguish generation-time dependencies from dependencies needed to
operate the completed rig afterward.

Identify where stock Rigify relies on:
- DEF-, ORG-, or MCH- prefixes.
- Side and numerical suffixes.
- Internal bone maps.
- Constraint subtargets.
- Driver paths.
- Widget names.
- Rigify-specific properties or registration state.

Follow the actual rig components and shared utilities invoked by
wolf.py, including body, limbs, paws, neck/head, and tail components.

## 10. Inspect the Blender references

First identify the armatures and record:
- Blender version.
- Object and datablock names.
- Object transforms.
- Bone counts and collections.
- Relevant scene dependencies.

Collect structured inspection data as needed:
- Bone names and parent relationships.
- Rest matrices, heads, tails, rolls, and connectivity.
- Deform flags and inheritance settings.
- Pose rotation modes.
- Constraints in order, with targets, subtargets, spaces, and settings.
- Drivers, expressions, variables, and target paths.
- Custom properties and UI metadata.
- Bone collections.
- Custom control shapes and relevant transforms.
- Relevant external objects, actions, and shape-key dependencies.

Inspect the body, custom face, and custom tail.

Save reusable structured snapshots in the repository and summarize
findings instead of flooding the conversation with raw data.

Keep the reference scene unchanged:
- Do not rename, delete, reparent, or regenerate bones.
- Do not overwrite the reference .blend.
- Keep pose and frame unchanged during inspection.
- Identify behavioral tests that should later run on a separate copy.

If MCP cannot retrieve required information, provide a targeted,
read-only Blender Python extraction script for the user to run.

## 11. Determine how fitting drives generation

The generator must adapt to a metarig fitted to a new mesh.

Investigate how reference structures can be derived from:
- Metarig bone transforms and lengths.
- Relative offsets in local bone coordinates.
- Ratios or explicit parameters.
- Relationships between multiple guide bones.

Pay particular attention to:
- Facial mechanisms and coordinated jaw motion.
- Custom tail controls and mechanisms.
- Body segment consolidation.
- Bone roll and constraint-space assumptions.
- Object-space versus armature-space transforms.

Identify the relationship between each relevant metarig bone and the
generated deform, mechanism, and control structures.

Separate extractable facts from authoring intent. Ask the user only
where intent remains ambiguous, such as:
- Which guide should drive a particular mechanism.
- Whether an offset should scale with anatomy.
- Whether a dependency is intentional or a manual-merge artifact.

A valid approach must reconstruct the rig from fitted guides, rather
than merely reproduce its original fixed coordinates.

## 12. Independent add-on feasibility

Compare:
- Vendoring the required Rigify generation infrastructure and rig types.
- Adapting selected components into a smaller dedicated generator.

Identify:
- Transitive imports.
- Rig registration and discovery assumptions.
- Namespace collisions.
- Runtime dependencies.
- Generated UI/script dependencies.
- Licensing and attribution requirements.

The result must operate without the installed Rigify add-on.

Give an evidence-based recommendation if possible. Reserve the detailed
implementation plan until important custom-rig unknowns are resolved.

## 13. Deliverables

Create investigation reports, structured snapshots, and any necessary
read-only extraction script in a dedicated research directory.

Leave implementation files and reference assets unchanged.

Produce a report containing:
1. Findings and architectural constraints.
2. Source-linked metarig creation trace.
3. Source-linked generation trace.
4. Wolf component/dependency map.
5. Inspection findings for Armature and Armature_original.
6. Custom face and tail reconstruction requirements.
7. Body consolidation and naming findings.
8. Independent reuse options and recommendation.
9. Bone correspondence and transform information relevant to future baking.
10. Remaining questions and targeted validation experiments.

For source claims, cite repository-relative paths, class/function names,
and relevant line numbers.

For scene claims, identify the armature, bone/property/constraint, and
saved snapshot supporting the finding.

Clearly distinguish:
- Verified source facts.
- Verified scene facts.
- Inferences.
- Unresolved authoring intent.
- Behavior requiring runtime testing.

End by stating whether the evidence is sufficient for planning.
If evidence is missing, list exactly what is needed and the smallest
next step to obtain it.

Complete the exploration and report without implementing the add-on.
1:29 AM
ok you write this:

The manually assembled animation rig consists of:
- A Rigify wolf body.
- The original head replaced with a custom facial rig.
- A custom tail.
- Additional facial bones and coordinated constraints for detailed
  deformation and easier animation.
so this is not corrent. Actually we want the process of the rig generation that apply to the wolf rig to be modified to generate our own custom rig. So claude will be confused with those instructions because he will look to copy those bones. But actually I inspected again the custom rig and its quite different. 

I am thinking let claude inspect rigify and all the functionality that are relevant to the wolf rigify rig and inspect the 2 blender rigs (armature and armature_original) and make his own plan on how to use the functionality from rigify and modify in order to create our custom rig. So the bone mapping would be only relevant in order to understand what bone from the wolf rig is represented in the custom rig, so he can map the controls correctly, although he might not need that because he will inspect the custom rig and the control bones have the same names as in rigify. 

Just a question why do you use opus in subagent B? Wont sonnet do the trick or even Haiku since its just reading code. This is only the investigation phase

So if you dont have any questions now with these comments recreate the prompt

…
12:19 PM
You’re right: the custom rig should define the target, and Rigify should supply reusable generation techniques. The mapping is supporting evidence, not instructions to copy or rename the wolf skeleton.

Sonnet is suitable for both exploration subagents. I suggested Opus for interpreting dependency and constraint interactions, but it isn’t necessary for this phase. Haiku could handle targeted searches; I’d use Sonnet for the broader investigation because it needs to connect findings across modules and the Blender scene.

Investigate the Rigify source and two reference Blender armatures to
determine how Rigify’s generation functionality can be reused and adapted
to generate our custom carnivore rig from our custom metarig.

Use two Sonnet exploration subagents for the independent source
investigations described below. The main agent should inspect Blender
and synthesize the findings.

This phase is investigation and an evidence-based architectural proposal.
Do not implement the add-on or modify the reference rigs.

## 1. Intended add-on workflow

Build an independent Blender add-on with this workflow:

1. A 3D Viewport sidebar panel contains a button named “carnivore”.
2. Clicking it creates our custom metarig.
3. The user positions the metarig bones to fit a new mesh.
4. A “Generate” button in the armature Object Data Properties generates
   our complete custom animation rig from the fitted metarig.

Target Blender 5.1+. Inspect the actual local Blender version and bundled
Rigify source to establish compatibility.

The add-on must operate without Rigify installed or enabled. Copying,
vendoring, or adapting Rigify code is acceptable. Investigate dependencies
and license/attribution obligations.

Current scope:
- Recreating the custom metarig.
- Generating the custom animation rig from fitted metarig bones.
- Adapting reusable Rigify generation functionality.
- Supporting the custom body, face, tail, controls, and mechanisms.

Future scope:
- Baking/transferring animation onto the metarig.
- Exporting the metarig as the game-engine skeleton.
- Export UI and engine integration.
- Automatic skinning and weight transfer.

Identify relevant future dependencies without implementing those features.

## 2. Source of truth and investigation approach

The custom animation rig is the target.

Do not assume it consists of a stock wolf body with a replacement face
and tail. Its structure differs from the stock wolf rig, and the extent
of those differences must be established by inspection.

Study the wolf generation pipeline as a source of reusable functionality:
rig types, generation stages, constraints, control construction,
mechanisms, utilities, and bookkeeping.

Then compare that functionality with the actual custom rig and determine
what can be reused, configured, adapted, or implemented specifically.

The goal is to adapt the generation process to produce our rig from our
metarig. Reproducing a stock wolf rig and applying a fixed list of bone
renames or deletions is not an assumed implementation strategy.

Use the evidence to recommend an approach. Reserve a detailed
implementation plan for the subsequent planning phase.

## 3. Available references

Rigify source is at the repository root:

rigify_original/

Primary entry point:

rigify_original/metarigs/Animals/wolf.py

Verify the actual path and capitalization.

The reference Blender file contains:

- Armature:
  Our custom metarig, containing the bones users position to fit a mesh.

- Armature_original:
  Our completed custom animation rig, which generation should reproduce.

The second object has also been referred to as “armature_original”.
Discover the actual case-sensitive names and distinguish object names
from armature datablock names.

A Blender MCP server and corresponding Blender add-on are available.
Discover the callable MCP capabilities and inspect the current scene.

If the reference file is not open, ask the user to open it or provide
its path. If armature identification is ambiguous, ask which is which.

Extract inspectable information automatically. Ask the user only about
missing assets or authoring intent that inspection cannot establish.

## 4. Rig architecture and future export

The generated animation rig may preserve the custom reference rig’s
existing hierarchy, including its control, mechanism, original, and
deform relationships.

The metarig supplies the clean hierarchy for future export.

The intended future workflow is:
1. Animate using the generated rig.
2. Bake/transfer animation onto the metarig.
3. Export the metarig.

Identify the correspondence and rest-transform information that future
animation transfer will need. Flag concrete incompatibilities without
implementing baking or redesigning the generated rig for export.

Do not assume matching names alone guarantees successful transfer.

## 5. Supporting wolf-to-custom correspondence

The following table is supporting information for understanding anatomical
and functional correspondence:

custom skeleton bone → source wolf bone(s)

root → root
pelvis → DEF-spine.004
spine_01 → DEF-spine.005
spine_02 → DEF-spine.006
spine_03 → DEF-spine.007
spine_04 → DEF-spine.008
neck_01 → DEF-spine.009
neck_02 → DEF-spine.010
head → DEF-spine.011

pelvis_l → DEF-pelvis.L
thigh_l → DEF-thigh.L, thigh.L.001
calf_l → DEF-shin.L, shin.L.001
knee_l → DEF-foot.L, foot.L.001
foot_l → DEF-toe.L

pelvis_r → DEF-pelvis.R
Remaining right hind-leg correspondences mirror the left.

clavicle_l → DEF-shoulder.L
upperarm_l → DEF-front_thigh.L, DEF-front_thigh.L.001
forearm_l → DEF-front_shin.L, DEF-front_shin.L.001
wrist_l → DEF-front_foot.L, DEF-front_foot.L.001
hand_l → DEF-front_toe.L

clavicle_r → DEF-shoulder.R
Remaining right foreleg correspondences mirror the left.

Some supplied source names omit DEF- on the second bone. Verify exact
names and roles against the source and scene.

This table is:
- A clue for relating custom anatomy to wolf generation components.
- Potential supporting evidence for understanding control correspondence.

It is not:
- A complete specification of the custom rig.
- A mandatory rename/delete procedure.
- A substitute for inspecting either armature.

The user reports that the custom control bones retain Rigify names;
verify this and use it where it helps establish correspondence.

Where a custom bone corresponds to two source segments, the intended
custom structure uses one bone spanning those segments. For example,
thigh_l spans from the beginning of the first thigh segment to the end
of the second. The user has tested this manually.

Inspect how the custom reference implements this and which controls and
mechanisms remain. Do not assume stock second-segment helpers are needed.

Use the custom reference and explicit target hierarchy to establish
what belongs in the target, rather than generating additional stock wolf
structures simply because they exist in Rigify.

## 6. Naming and metarig hierarchy

Use base_side_number when both side and number are present, for example:

thigh_l_01

Preserve explicitly specified names without numerical suffixes,
including thigh_l, calf_l, upperarm_l, forearm_l, wrist_l, and hand_l,
with their _r counterparts.

Preserve facial spelling and capitalization, including teethUpper,
lipUpper_l_01, and CTR_wrinkles.

The following is the intended metarig / future export hierarchy.
The generated animation rig may retain its own internal hierarchy.

root
└── pelvis
    ├── tail_01
    │   └── tail_02
    │       └── tail_03
    │           └── tail_04
    │               └── tail_05
    │                   └── tail_06
    │                       └── tail_07
    │                           └── tail_08
    │                               └── tail_09
    │                                   └── tail_10
    │                                       └── tail_11
    ├── pelvis_l
    │   └── thigh_l
    │       └── calf_l
    │           └── knee_l
    │               └── foot_l
    ├── pelvis_r
    │   └── thigh_r
    │       └── calf_r
    │           └── knee_r
    │               └── foot_r
    └── spine_01
        └── spine_02
            └── spine_03
                └── spine_04
                    ├── neck_01
                    │   └── neck_02
                    │       └── head
                    │           ├── nose_01
                    │           │   └── nose_02
                    │           │       └── nose_03
                    │           │           └── nose_04
                    │           │               ├── nose_05
                    │           │               │   ├── nose_06
                    │           │               │   ├── nose_l_07
                    │           │               │   │   └── nose_l_08
                    │           │               │   └── nose_r_07
                    │           │               │       └── nose_r_08
                    │           │               ├── nose_l_05
                    │           │               │   └── nose_l_06
                    │           │               └── nose_r_05
                    │           │                   └── nose_r_06
                    │           ├── eyelid_l_01
                    │           │   └── eyelid_l_02
                    │           │       └── eyelid_l_03
                    │           │           └── eyelid_l_04
                    │           │               └── eyelid_l_05
                    │           │                   └── eyelid_l_06
                    │           │                       └── eyelid_l_07
                    │           │                           └── eyelid_l_08
                    │           ├── eyebrow_l_01
                    │           │   └── eyebrow_l_02
                    │           │       └── eyebrow_l_03
                    │           │           └── eyebrow_l_04
                    │           │               └── eyebrow_l_05
                    │           │                   └── eyebrow_l_06
                    │           │                       └── eyebrow_l_07
                    │           │                           └── eyebrow_l_08
                    │           │                               └── nose_l_01
                    │           │                                   └── nose_l_02
                    │           │                                       └── nose_l_03
                    │           │                                           └── nose_l_04
                    │           ├── teethUpper
                    │           ├── eye_l
                    │           ├── cheek_l_01
                    │           │   ├── cheek_l_02
                    │           │   │   └── brow_l_01
                    │           │   │       └── brow_l_02
                    │           │   │           └── brow_l_03
                    │           │   │               └── brow_l_04
                    │           │   ├── mouth_l_01
                    │           │   │   └── mouth_l_02
                    │           │   │       └── mouth_l_03
                    │           │   │           └── mouth_l_04
                    │           │   ├── lipUpper_l_01
                    │           │   │   └── lipUpper_l_02
                    │           │   │       └── lipUpper_l_03
                    │           │   │           └── lipUpper_l_04
                    │           │   │               └── lipUpper_l_05
                    │           │   └── lipLower_l_01
                    │           │       └── lipLower_l_02
                    │           │           └── lipLower_l_03
                    │           │               └── lipLower_l_04
                    │           │                   └── lipLower_l_05
                    │           ├── jawbone
                    │           │   ├── face_l_01
                    │           │   │   └── face_l_02
                    │           │   │       └── face_l_03
                    │           │   │           └── face_l_04
                    │           │   ├── chin_01
                    │           │   │   └── chin_02
                    │           │   │       └── chin_03
                    │           │   │           └── chin_04
                    │           │   ├── teethLower
                    │           │   ├── tongue
                    │           │   │   └── tongue_01
                    │           │   │       └── tongue_02
                    │           │   │           └── tongue_03
                    │           │   │               └── tongue_04
                    │           │   └── face_r_01
                    │           │       └── face_r_02
                    │           │           └── face_r_03
                    │           │               └── face_r_04
                    │           ├── eyelid_r_01
                    │           │   └── eyelid_r_02
                    │           │       └── eyelid_r_03
                    │           │           └── eyelid_r_04
                    │           │               └── eyelid_r_05
                    │           │                   └── eyelid_r_06
                    │           │                       └── eyelid_r_07
                    │           │                           └── eyelid_r_08
                    │           ├── eyebrow_r_01
                    │           │   └── eyebrow_r_02
                    │           │       └── eyebrow_r_03
                    │           │           └── eyebrow_r_04
                    │           │               └── eyebrow_r_05
                    │           │                   └── eyebrow_r_06
                    │           │                       └── eyebrow_r_07
                    │           │                           └── eyebrow_r_08
                    │           │                               └── nose_r_01
                    │           │                                   └── nose_r_02
                    │           │                                       └── nose_r_03
                    │           │                                           └── nose_r_04
                    │           ├── eye_r
                    │           ├── cheek_r_01
                    │           │   ├── cheek_r_02
                    │           │   │   └── brow_r_01
                    │           │   │       └── brow_r_02
                    │           │   │           └── brow_r_03
                    │           │   │               └── brow_r_04
                    │           │   ├── mouth_r_01
                    │           │   │   └── mouth_r_02
                    │           │   │       └── mouth_r_03
                    │           │   │           └── mouth_r_04
                    │           │   ├── lipUpper_r_01
                    │           │   │   └── lipUpper_r_02
                    │           │   │       └── lipUpper_r_03
                    │           │   │           └── lipUpper_r_04
                    │           │   │               └── lipUpper_r_05
                    │           │   └── lipLower_r_01
                    │           │       └── lipLower_r_02
                    │           │           └── lipLower_r_03
                    │           │               └── lipLower_r_04
                    │           │                   └── lipLower_r_05
                    │           ├── CTR_wrinkles
                    │           ├── ear_l_01
                    │           │   └── ear_l_02
                    │           └── ear_r_01
                    │               └── ear_r_02
                    ├── clavicle_l
                    │   └── upperarm_l
                    │       └── forearm_l
                    │           └── wrist_l
                    │               └── hand_l
                    └── clavicle_r
                        └── upperarm_r
                            └── forearm_r
                                └── wrist_r
                                    └── hand_r

Inspect the metarig against this hierarchy and report discrepancies.

CTR_wrinkles is explicitly included. Inspect its role and dependencies;
do not classify or discard it solely by its prefix.

## 7. Investigation assignments

Run these two source investigations in parallel using Sonnet.

### Subagent A: generation infrastructure

Trace:
- Wolf metarig creation and registration.
- UI/operator entry points.
- Rig-type discovery and instantiation.
- Generation stages and their ordering.
- Shared generation infrastructure and dependencies.
- Generated UI and runtime requirements.
- Regeneration and association with the source metarig.

### Subagent B: reusable rig functionality

Trace:
- The actual rig types used by the wolf.
- Spine, limbs, paws, neck/head, and tail functionality.
- Control and mechanism construction.
- Constraints, drivers, and inter-component dependencies.
- Naming and parenting assumptions.
- Configuration and extension points.
- Which functionality could support the custom rig, once scene evidence
  is available.

The main agent:
- Owns the Blender MCP connection.
- Inspects both reference armatures.
- Saves structured snapshots.
- Shares relevant scene findings with the source subagents.
- Compares the custom rig with the reusable Rigify functionality.
- Produces the combined report and architectural recommendation.

Give subagents the relevant requirements explicitly. Keep all live
Blender interaction with the main agent to avoid shared-scene conflicts.

If the requested model or subagent capabilities are unavailable, state
the limitation and continue with available capabilities.

## 8. Trace metarig creation and rig generation

Follow wolf.py into the modules it actually uses.

For metarig creation, establish:
- How bones, transforms, rolls, parenting, and collections are created.
- How rig types and generation parameters are assigned.
- What metadata is necessary for generation.
- How our custom metarig could be recreated through Python.
- How the creation operation could support the “carnivore” sidebar button.

For rig generation, establish:
- Generate operator entry point and validation.
- Reading of metarig structure and parameters.
- Rig-type construction and generation lifecycle.
- Creation of original, deform, mechanism, and control bones.
- Parenting, constraints, drivers, and custom properties.
- Control shapes and bone collections.
- Generated UI and scripts.
- Source/generated-rig association and regeneration behavior.

Distinguish generation-time dependencies from dependencies required to
operate the finished rig.

Identify assumptions involving:
- Bone prefixes, side suffixes, and numerical suffixes.
- Internal bone correspondence maps.
- Constraint targets and spaces.
- Driver paths and custom properties.
- Widget names.
- Rigify registration and module namespaces.

Cite actual implementation rather than relying on general familiarity
with Rigify.

## 9. Inspect both Blender armatures

Discover the available MCP commands first.

Record:
- Blender version.
- Object and armature datablock identities.
- Object transforms.
- Bone counts and collections.
- Relevant scene dependencies.

Extract structured data as needed:
- Bone names, parents, connectivity, and deform flags.
- Rest matrices, heads, tails, and rolls.
- Inheritance settings and pose rotation modes.
- Constraints in order, including targets, subtargets, spaces,
  influences, and other relevant settings.
- Drivers, expressions, variables, and target paths.
- Custom properties and UI metadata.
- Bone collections.
- Custom control shapes and transforms.
- Relevant external objects, actions, and shape-key relationships.
- Available generation metadata on the metarig.

Inspect the complete custom rig, including body, face, and tail.

One known facial behavior is that opening the jaw also influences the
upper mouth, nose, and eyebrows. Determine how this is implemented,
rather than assuming a particular constraint or driver setup.

Save structured snapshots in a dedicated repository research directory.
Summarize the findings in conversation instead of dumping all raw data.

Keep reference assets unchanged:
- Do not rename, reparent, delete, or regenerate bones.
- Do not overwrite the reference .blend.
- Keep pose and frame unchanged during read-only inspection.
- Identify behavioral tests that should later run on a separate copy.

If MCP cannot retrieve necessary data, provide a targeted read-only
Blender Python extraction script for the user to run.

## 10. Compare the custom rig with Rigify functionality

Produce a component-level comparison grounded in source and scene data.

For each significant custom system, establish:
- Its metarig inputs.
- Its generated bones and their roles.
- Its controls, constraints, drivers, and properties.
- Its dependencies on other systems.
- Any corresponding Rigify functionality.
- Whether that functionality appears reusable as-is, configurable,
  adaptable, or unsuitable.
- What evidence is still missing.

Use matching control names and the supplied correspondence table as
clues. Verify functional equivalence before concluding that a custom
system can use a stock rig type.

Explain the custom rig’s differences from stock wolf generation rather
than treating those differences as errors to normalize away.

## 11. Determine how fitted guides drive generation

The result must adapt when the metarig is fitted to a new animal.

Investigate how generated structures can be derived from:
- Guide-bone transforms and lengths.
- Relative offsets in local coordinates.
- Ratios and explicit parameters.
- Relationships between multiple guides.
- Bone roll and orientation.
- Object-space and armature-space transforms.

Pay particular attention to facial mechanisms, the custom tail, and
custom body structures.

Identify how to preserve control behavior while adapting positions and
proportions. Copying fixed reference coordinates is insufficient.

Separate extractable facts from authoring intent. Ask the user only
where inspection cannot resolve questions such as:
- Which guide determines a particular mechanism.
- Whether an offset should scale with anatomy.
- Whether a relationship is intentional or a reference-rig artifact.

## 12. Independent add-on architecture

Based on the evidence, assess:
- Vendoring the required Rigify infrastructure and adapting rig types.
- Reusing selected Rigify components within a dedicated generator.
- Any other approach clearly supported by the findings.

For each viable option, identify:
- Required modules and transitive dependencies.
- Registration and discovery assumptions.
- Namespace considerations.
- Runtime and generated-script dependencies.
- Licensing and attribution obligations.
- How custom metarig data would feed generation.
- How closely it can reproduce the reference behavior.
- Main uncertainties and validation needs.

Recommend an approach with reasons. Keep this at the architectural level
so a subsequent planning phase can turn verified findings into concrete
implementation steps.

## 13. Deliverables

You may create reports, structured inspection snapshots, and an optional
read-only extraction script in a dedicated research directory.

Leave add-on implementation files and reference assets unchanged.

Produce a report containing:
1. Findings and architectural constraints.
2. Source-linked metarig creation trace.
3. Source-linked generation lifecycle.
4. Wolf component and dependency map.
5. Inspection findings for both custom armatures.
6. Custom-versus-Rigify component comparison.
7. Proposed derivation of generated structures from fitted guides.
8. Independent add-on options and architectural recommendation.
9. Correspondence information relevant to future animation baking.
10. Remaining questions and focused validation experiments.

For source claims, cite repository-relative paths, class/function names,
and relevant line numbers.

For scene claims, cite the armature, relevant bones/constraints/properties,
and the saved inspection snapshot.

Clearly distinguish:
- Verified source facts.
- Verified scene facts.
- Inferences.
- Unresolved authoring intent.
- Behavior requiring runtime testing.

End by stating whether the evidence is sufficient for detailed planning.
For each remaining gap, specify the smallest next step needed to resolve it.

Complete the investigation and report without implementing the add-on.
12:19 PM











