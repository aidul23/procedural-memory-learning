It walks through how to give an RL-style agent **procedural memory**: it learns reusable skills from experience, stores them as modules, and then reuses them to solve future episodes faster and with higher reward.[1][2]

## What they tried to do

- **Model skills as “neural modules”**  
  - Each `Skill` stores: a name, preconditions on state, an action sequence, an embedding summarizing context + actions, and usage/success stats.[3][1]
  - A `SkillLibrary` maintains a list of skills, merges very-similar ones (by cosine similarity), and retrieves the best skills for a given state.[1][3]

- **Use a simple GridWorld as a testbed**  
  - Environment: 5×5 grid, you start at (0,0), must pick up a key, open a door, then reach the goal at bottom-right.[3]
  - Reward structure encourages the key→door→goal sequence and penalizes wandering with small negative step cost.[3]

- **Let procedural memory emerge from trajectories**  
  - The agent begins with only primitive actions (`move_*`, `pickup_key`, `open_door`) and a handcrafted exploration policy (`_choose_exploration_action`).[1][3]
  - When an episode succeeds, it slices off a short successful segment (last few steps), turns it into a `Skill`, computes an embedding, and adds it to the library.[1][3]
  - On later episodes, before exploring, it queries the skill library with the current state embedding and tries the best-matching skill first.[2][3]

## How the system works step by step

- **Skill representation and storage**  
  - `Skill.is_applicable(state)` checks simple symbolic preconditions like `has_key` or `door_open`.[3]
  - Embeddings encode: agent position (hashed), key/door flags, and a hashed summary of the action sequence, then normalized; similarity is cosine.[3]
  - When adding a new skill, if its embedding is very similar to an existing skill (similarity > 0.9), they just bump that skill’s success count instead of storing a duplicate.[3]

- **Exploration vs skill use**  
  - `_choose_exploration_action` is a scripted “go to key → go to door → go to goal” policy; it’s not optimal but ensures eventual success.[1][3]
  - `run_episode` loop:  
    - If skills exist, compute a query embedding for the current state, retrieve top-1, and execute its full action sequence.  
    - If no applicable skill or none retrieved, fall back to `primitive_actions` via `_choose_exploration_action`.[1][3]

- **Training and emergence of procedural memory**  
  - `train(episodes)` runs many episodes, logs rewards, steps, number of skills, and total skill uses.[1][3]
  - On successful episodes, it extracts skills from the last few transitions, so things like “acquire_key”, “open_door_sequence”, or generic navigation chains are gradually learned.[3]
  - Visualization (`visualize_training`) shows that as the skill library grows, steps per episode typically drop and rewards improve, demonstrating benefit of reuse.[2][3]

## What you can achieve with this pattern

- **Concrete behaviors in this toy setup**  
  - See the agent go from “clumsy” scripted movement to reusing skills like “go to key and pick it up” or “open the door” as single callable modules.[3]
  - Empirically demonstrate that procedural memory (skills) reduces sample complexity: fewer steps, faster convergence to high reward.[2][3]

- **As a template for your own work**  
  - Replace `GridWorld` with your own environment (e.g., a tool-using LLM agent, a workflow engine, or a robotics sim).  
  - Define **skill extraction** rules for your domain (what counts as a useful reusable segment of a trajectory?).  
  - Tune **preconditions** and **embeddings** to capture the right context (e.g., user profile, task description, tool states) so skills generalize beyond exact repeats.[4][5]

- **Research or project directions you can explore**  
  - Compare this simple trajectory-slicing approach vs RL-based options discovery or hierarchical RL (options, skills, sub-policies).  
  - Extend skill updating: decay or delete rarely-successful skills, split or refine skills when they fail, track success by context cluster.  
  - Integrate with **episodic** memory (raw trajectories) and **semantic** memory (abstract knowledge) to get a more complete cognitive agent stack.[6][5]


[1](https://www.marktechpost.com/2025/12/09/a-coding-guide-to-build-a-procedural-memory-agent-that-learns-stores-retrieves-and-reuses-skills-as-neural-modules-over-time/)
[2](https://www.kiadev.net/news/2025-12-09-coding-guide-procedural-memory-agent)
[3](https://www.marktechpost.com/2025/08/19/memp-a-task-agnostic-framework-that-elevates-procedural-memory-to-a-core-optimization-target-in-llm-based-agent/)
[4](https://machinelearningmastery.com/beyond-short-term-memory-the-3-types-of-long-term-memory-ai-agents-need/)
[5](https://www.marktechpost.com/2025/11/15/how-to-build-memory-powered-agentic-ai-that-learns-continuously-through-episodic-experiences-and-semantic-patterns-for-long-term-autonomy/)
[6](https://nielsberglund.com/post/2025-12-14-interesting-stuff---week-50-2025/)
[7](https://muckrack.com/asifrazzaq/articles)
[8](https://www.facebook.com/groups/MontrealAI/posts/1412450992550062/)
[9](https://techling.ai/blog/memory-agent-neural-modules-guide/)
[10](https://www.instagram.com/p/DS-Usqsj-wV/)
[11](https://www.facebook.com/groups/RealAGI/posts/3110485185826727/)
[12](https://www.linkedin.com/posts/nextbusiness24_a-coding-information-to-construct-a-procedural-activity-7404300073813311491-XN7T)
[13](https://www.ruh.ai/blogs/ai-agent-memory-systems)
[14](https://arxiv.org/html/2505.03434v1)
[15](https://arxiv.org/html/2512.10696v1)