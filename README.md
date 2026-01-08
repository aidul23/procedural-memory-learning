It walks through how to give an RL-style agent **procedural memory**: it learns reusable skills from experience, stores them as modules, and then reuses them to solve future episodes faster and with higher reward.

## What they tried to do

- **Model skills as “neural modules”**  
  - Each `Skill` stores: a name, preconditions on state, an action sequence, an embedding summarizing context + actions, and usage/success stats.
  - A `SkillLibrary` maintains a list of skills, merges very-similar ones (by cosine similarity), and retrieves the best skills for a given state.

- **Use a simple GridWorld as a testbed**  
  - Environment: 5×5 grid, you start at (0,0), must pick up a key, open a door, then reach the goal at bottom-right.
  - Reward structure encourages the key→door→goal sequence and penalizes wandering with small negative step cost.

- **Let procedural memory emerge from trajectories**  
  - The agent begins with only primitive actions (`move_*`, `pickup_key`, `open_door`) and a handcrafted exploration policy (`_choose_exploration_action`).
  - When an episode succeeds, it slices off a short successful segment (last few steps), turns it into a `Skill`, computes an embedding, and adds it to the library.
  - On later episodes, before exploring, it queries the skill library with the current state embedding and tries the best-matching skill first.

## How the system works step by step

- **Skill representation and storage**  
  - `Skill.is_applicable(state)` checks simple symbolic preconditions like `has_key` or `door_open`.
  - Embeddings encode: agent position (hashed), key/door flags, and a hashed summary of the action sequence, then normalized; similarity is cosine.
  - When adding a new skill, if its embedding is very similar to an existing skill (similarity > 0.9), they just bump that skill’s success count instead of storing a duplicate.

- **Exploration vs skill use**  
  - `_choose_exploration_action` is a scripted “go to key → go to door → go to goal” policy; it’s not optimal but ensures eventual success.
  - `run_episode` loop:  
    - If skills exist, compute a query embedding for the current state, retrieve top-1, and execute its full action sequence.  
    - If no applicable skill or none retrieved, fall back to `primitive_actions` via `_choose_exploration_action`.

- **Training and emergence of procedural memory**  
  - On successful episodes, it extracts skills from the last few transitions, so things like “acquire_key”, “open_door_sequence”, or generic navigation chains are gradually learned.
  - Visualization (`visualize_training`) shows that as the skill library grows, steps per episode typically drop and rewards improve, demonstrating the benefit of reuse.

## What you can achieve with this pattern

- **Concrete behaviors in this toy setup**  
  - See the agent go from “clumsy” scripted movement to reusing skills like “go to key and pick it up” or “open the door” as single callable modules.
  - Empirically demonstrate that procedural memory (skills) reduces sample complexity: fewer steps, faster convergence to high reward.

- **As a template for your own work**  
  - Replace `GridWorld` with your own environment (e.g., a tool-using LLM agent, a workflow engine, or a robotics sim).  
  - Define **skill extraction** rules for your domain (what counts as a useful, reusable segment of a trajectory?).  
  - Tune **preconditions** and **embeddings** to capture the right context (e.g., user profile, task description, tool states) so skills generalize beyond exact repeats.

- **Research or project directions you can explore**  
  - Compare this simple trajectory-slicing approach vs RL-based options discovery or hierarchical RL (options, skills, sub-policies).  
  - Extend skill updating: decay or delete rarely-successful skills, split or refine skills when they fail, track success by context cluster.  
  - Integrate with **episodic** memory (raw trajectories) and **semantic** memory (abstract knowledge) to get a more complete cognitive agent stack.
