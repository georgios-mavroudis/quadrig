Use the explorer subagent to investigate the local Rigify source and produce a report for a subsequent planning phase.

This is an investigation task only. Do not implement the add-on, modify existing files, or produce an implementation plan prematurely. You may propose targeted inspection scripts if information about my custom rig is missing.

## Project objective

Build an independent Blender add-on for advanced carnivore quadruped rigs.

The user workflow must be:

1. In a 3D Viewport sidebar panel, click a button named “carnivore”.
2. The add-on creates my custom metarig with my exact guide-bone names.
3. The user positions the guide bones to fit their mesh.
4. In the armature Object Data Properties, click “Generate”.
5. The add-on generates the complete body and advanced facial rig.

Export tools, mesh binding, and automatic weight painting are outside the current scope.

Target Blender 5.1 or newer. Verify the actual local Blender and bundled Rigify versions when possible; do not assume API compatibility or release/LTS status from version labels alone.

The add-on must work without Rigify installed or enabled. Reusing or adapting Rigify code is acceptable. Investigate the licensing, attribution, and technical implications of doing so.

## Source and reference assets

Rigify source is in the repository root:

rigify_original/

The primary reference is:

rigify_original/metarigs/Animals/wolf.py

Verify the actual path and capitalization.

My custom metarig already exists and contains the intended anatomical guide bones and names. It is not something you should invent from scratch.

Additional inputs:

- Custom metarig location or description:
  YOU_CUSTOM_METARIG_REFERENCE

- Completed custom rig location, or “inspection output to be supplied later”:
  YOU_CUSTOM_RIG_REFERENCE

- Intended guide-bone names, or location of the list:
  YOU_GUIDE_BONE_NAMES

- Existing deform-bone names / old-to-new mapping, if available:
  YOU_DEFORM_BONE_NAMES_OR_MAPPING

- Required deform hierarchy, if available:
  YOU_DEFORM_HIERARCHY

Do not block the Rigify investigation if these inputs are incomplete. Explicitly record what is missing.

## Existing rig and intended behavior

My current manually assembled rig consists of:

- The Rigify wolf body.
- The original head removed and replaced with my custom facial rig.
- A more detailed facial bone setup for realistic deformation.
- Constraints and coordinated facial motion that simplify animation.

For example, opening the jaw also influences the upper mouth, nose, and eyebrows. The exact mechanisms, influence values, and relationships must be inspected; do not infer them from this description.

The current completed rig is a reference for controls and deformation behavior, NOT a reference for the desired final bone hierarchy.

Its hierarchy currently has the same shortcomings I encounter with the Rigify output. My current game-engine workaround is:

1. Animate with the full control rig.
2. Bake animation onto deform bones.
3. Use another skeleton with the mechanism, original, and control bones removed.
4. Reparent the remaining deform bones into a clean hierarchy.
5. Export that skeleton and mesh.

The new add-on should eliminate the need to reconstruct the deform hierarchy later.

## Required generated deform hierarchy

A clean deform/export skeleton must exist INSIDE the generated animation rig.

Requirements:

- One designated export root.
- Every deform bone descends from that root.
- Anatomically appropriate parenting for torso, limbs, neck, head, face, and tail.
- No mechanism, original, or control bones in the ancestor paths between export-skeleton bones.
- Removing helper bones must leave the intended skeleton hierarchy intact.
- Preserve the intended animation controls and deformation behavior.

The export root may be a dedicated non-deforming bone retained as part of the export skeleton. Distinguish this from an animator-facing root control.

Treat this architecture as a requirement. Investigate feasibility and technical conflicts. Do not silently substitute a separate export skeleton as the recommended implementation.

Do not assume changing parents is harmless. Investigate effects on local transforms, rest matrices, constraint spaces, scale inheritance, drivers, and generation-time assumptions.

This does not require implementing export now, nor does it imply that deleting helpers alone preserves animation without baking.

## Naming requirements

My deform bones have already been renamed to my intended Unreal-style convention.

Example:

DEF-thigh.L.001 → thigh_01_l

Use my actual mapping or bone list as the authority. This example is not a complete renaming algorithm, and it is not a claim that Unreal requires these exact names.

Investigate:

- Where Rigify assumes DEF-, ORG-, MCH-, side suffixes, or numerical suffixes.
- Where generated names are referenced by constraints, drivers, properties, widgets, animation paths, and generation bookkeeping.
- Whether deform-bone identification can be independent of a DEF- prefix.
- How guide bones should map to generated bones without ambiguous string substitutions.

## Investigation questions

### 1. Metarig creation

Trace the wolf metarig from its UI/operator entry point through creation.

Explain:

- How the metarig is registered and exposed in the UI.
- How bones, transforms, parenting, and bone collections are created.
- How rig types and parameters are assigned.
- Which information in wolf.py controls later generation.
- Which supporting modules and rig types it relies on.
- How the custom guide names and an existing custom metarig could fit this approach.

### 2. Rig generation

Trace the complete path from clicking Generate to the finished rig.

Explain:

- Operator entry points and validation.
- Metarig discovery and rig-type instantiation.
- Generation stages and their ordering.
- Creation of original, deform, mechanism, and control bones.
- Parenting, constraints, drivers, custom properties, and control shapes.
- Generated UI and runtime dependencies.
- How the generated rig is related to the source metarig.
- Existing behavior around generating again and replacing/updating an existing rig.

Distinguish what runs during generation from what must remain available afterward.

### 3. Wolf body dependencies

Identify the actual rig components used by the wolf and trace their dependencies.

Cover the spine, limbs, paws, neck/head, and tail as present in the source.

Explain:

- Which components are reusable for the body.
- How head/neck generation interacts with the torso.
- Where the stock head could be replaced by a custom face system.
- Which naming, parenting, and parameter assumptions cross component boundaries.

Follow dependencies beyond wolf.py. Do not stop at a high-level summary of that file.

### 4. Clean deform hierarchy

Determine how the current wolf generation constructs the actual deform hierarchy, using source evidence.

Distinguish bone parenting from constraint relationships and collection membership.

Identify:

- Where deform parenting is assigned or overwritten.
- Why deform chains are separated, if they are.
- Which behaviors depend on that structure.
- Where generation could produce the required clean hierarchy.
- What additional constraints or transform handling might be needed to preserve behavior.
- What requires a Blender experiment to verify.

Do not declare compatibility proven by static source inspection alone.

### 5. Independent add-on boundaries

Investigate the reusable subset of Rigify needed for this scope.

Compare, at an exploratory level:

- Vendoring the necessary Rigify generation infrastructure and rig types.
- Adapting selected components into a smaller dedicated generator.

Identify transitive dependencies, registration assumptions, module namespace issues, generated-script dependencies, and licensing obligations.

Give an evidence-based recommendation if possible, but reserve a concrete implementation plan until the custom rig has been inspected.

### 6. Custom face and metarig inspection

Do not invent my facial rig implementation.

Explain what information is needed to reconstruct it procedurally from the custom guides.

If direct inspection is unavailable, propose a read-only Blender Python extraction script for me to run. It should collect the relevant data in a structured format, including as needed:

- Blender version and armature identities.
- Bone names, parent relationships, rest transforms, and deform flags.
- Pose-bone rotation modes and relevant inheritance settings.
- Constraints, ordering, targets, subtargets, spaces, and influences.
- Drivers, expressions, variables, and target paths.
- Custom properties and relevant UI metadata.
- Bone collections and memberships.
- Custom control-shape references and transforms.
- Relevant relationships to mesh shape keys or other objects, if present.

Distinguish data that can be extracted automatically from authoring intent I need to explain, especially:
- Which guide controls each generated structure.
- Which offsets should scale with anatomy.
- Which relationships are deliberate.
- Which parts of the current rig are artifacts of the manual merge.

Request only the missing information necessary for the next investigation or planning phase. Do not require a .blend file if structured extraction can answer the questions.

## Report requirements

Produce a report suitable for handing to a planning agent.

Include:

1. Findings and major architectural constraints.
2. A source-linked trace of metarig creation.
3. A source-linked trace of rig generation.
4. The wolf’s component/dependency map.
5. Findings about clean deform parenting and custom naming.
6. Independence/reuse options and their tradeoffs.
7. Missing custom-rig evidence and proposed inspection steps.
8. Concrete questions or experiments needed before planning.

For important claims, cite repository-relative file paths, function/class names, and relevant line numbers.

Clearly separate:
- Verified source findings.
- Inferences.
- Unverified assumptions.
- Questions requiring my rig data or Blender runtime testing.

Keep the investigation focused on the carnivore workflow. Do not implement anything yet.