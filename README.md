# AVMA-CVRP

AVMA-CVRP is an adaptive visual multi-agent framework for the Capacitated Vehicle Routing Problem (CVRP). Multimodal agents reason from rendered problem and route images, while a deterministic Python observer validates feasibility, evaluates route quality, and coordinates the search without exposing hidden numerical problem data to the agents.

## System architecture

The system separates visual reasoning from deterministic control:

- The visual layer renders CVRP instances and candidate routes using configurable demand encodings.
- Specialized multimodal agents propose, compare, repair, and diversify route sets.
- The deterministic observer parses CVRPLIB instances, validates capacity and coverage constraints, evaluates distances, detects structural stagnation, and manages state transitions.
- Provider adapters connect the protocol to supported multimodal model APIs.
- Versioned configurations, prompts, and scale policies make the protocol reproducible.

The main entry point is `run_adaptive_multi_agent.py`. Core modules are located under `src/`.

## Agent roles

- **Initializer** creates the first complete route proposal from the problem image.
- **Critic** proposes alternative route sets for the current search state.
- **Scorer** ranks candidate images using visual evidence.
- **Repair** attempts to restore feasibility when a selected route set is invalid.
- **Hybrid** applies an LLM-guided intra-route improvement after structural stagnation.
- **Diversity** generates a fresh route proposal when additional exploration is needed.

Agent instructions are stored in `prompts/cvrp_capacity_v3/`.

## Workflow

1. Load a CVRPLIB instance and an experiment configuration.
2. Apply the versioned workspace scale policy.
3. Render the model-facing problem image.
4. Ask the Initializer for a complete route set.
5. Validate proposals with the deterministic observer.
6. Iterate through Critic, Scorer, and Repair stages.
7. Detect structural stagnation and activate Hybrid or Diversity behavior when required.
8. Maintain protocol state and provenance throughout execution.

## Information firewall

The multimodal agents may use only visible information and their role instructions, including visible customer locations, node labels, the depot marker, route connections, demand encodings, and visible vehicle symbols.

The agents are not given exact coordinates, distance matrices, numerical demands, numerical capacities, route loads, numerical route lengths, feasibility diagnostics, reference solutions, or optimality information. These values remain inside the deterministic observer and are used only for validation and control.

## Visual demand encodings

The repository provides four visual demand encodings:

- Node size
- Bar length
- Dot density
- Color intensity

Each encoding supports collision-aware placement and side-panel placement. The corresponding YAML configurations are stored under `configs/main_8method/`.

## Installation

Python 3.10 or newer is recommended.

```bash
python -m venv .venv
```

Activate the environment using the command appropriate for your platform, then install the dependencies:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

For API-backed execution, define the key required by the selected provider in a local `.env` file:

```text
GEMINI_API_KEY=
GROQ_API_KEY=
MISTRAL_API_KEY=
OPENROUTER_API_KEY=
COHERE_API_KEY=
```

Only the key for the selected provider is required. Never commit credentials.

## Configuration

Protocol configurations are stored in `configs/main_8method/`. Each YAML file defines the visual encoding, placement strategy, rendering settings, agent parameters, provider defaults, and search controls.

### Experiment reproducibility

The eight main visual-method configurations specify a maximum of **20 search iterations** and use the model identifier `gemini-3.7-flash`, as recorded for the study. Runs may complete fewer iterations due to early stopping or the failure to establish a feasible initial solution. Provider model availability and routing can change over time; exact reproduction should be checked against the original execution metadata and the provider's current model identifiers.

The active prompt set is `cvrp_capacity_v3`. The default scale policy is:

```text
data/cvrplib/scale_policies/benchmark_scale_v1.json
```

The command line must provide a provider and a vision-capable model. Command-line values override the provider and model fields in the selected YAML file.

## Usage

Run commands from the repository root.

Validate the configuration, instance, prompts, and rendering path without making an API request:

```bash
python run_adaptive_multi_agent.py \
  --instance data/cvrplib/P-n21-k2.vrp \
  --config configs/main_8method/size_collision.yaml \
  --provider gemini \
  --model YOUR_VISION_MODEL \
  --max-vehicles 2 \
  --validate-only
```

Start an API-backed search by removing `--validate-only` after configuring the selected provider:

```bash
python run_adaptive_multi_agent.py \
  --instance data/cvrplib/P-n21-k2.vrp \
  --config configs/main_8method/size_collision.yaml \
  --provider gemini \
  --model YOUR_VISION_MODEL \
  --max-vehicles 2
```

Use a different YAML file to select another visual encoding and placement strategy.

## Testing

Run the test suite from the repository root:

```bash
python -m pytest -q
```

The tests cover configuration loading, CVRP validation, hybrid transitions, deterministic rendering, workspace scaling, and search utilities.

## Citation

If you use AVMA-CVRP in academic work, cite the associated manuscript:

> Nimet Asude Yalçın, Ayşe Nihal Kaymak, and Furkan Yener. *AVMA-CVRP: An Adaptive Visual Multi-Agent System for the Capacitated Vehicle Routing Problem*.

Machine-readable citation metadata is provided in `CITATION.cff`.
