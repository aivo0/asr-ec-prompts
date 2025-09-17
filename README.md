# ASR Error Correction Prompts

This repository contains the prompts used in the paper "When Less is More: Simple Prompting Outperforms Complex Multi-Agent Systems for ASR Error Correction" (submitted to ICASSP 2026) for Estonian and Ukrainian ASR error correction experiments.

## Overview

This work compares three approaches for LLM-based ASR error correction:
1. **Baseline Single-Hypothesis Correction** (Cautious and Permissive strategies)
2. **Multi-Agent Systems** with ReAct prompting
3. **Reasoning Models** with Constrained Cautious strategy

## Repository Structure

```
.
├── estonian/
│   ├── cautious.txt                    # Conservative baseline (±5% length, minimal corrections)
│   ├── permissive.txt                  # Liberal baseline (broader grammatical improvements)
│   ├── constrained_cautious.txt        # For Gemini 2.5 Pro reasoning model (batch processing)
│   └── multi_agent/
│       ├── analysis_agent.txt          # Error detection and hypothesis generation
│       ├── search_agent.txt            # Entity verification via web search
│       └── synthesis_agent.txt         # Final correction decisions
│
├── ukrainian/
│   ├── constrained_cautious.txt        # For Gemini 2.5 Pro reasoning model (batch processing)
│   └── multi_agent/
│       ├── analysis_agent.txt          # Ukrainian error detection
│       ├── search_agent.txt            # Ukrainian entity verification
│       └── synthesis_agent.txt         # Ukrainian final corrections
│
├── global_context_summary.txt          # Optional: Global context summarization (tested but no improvement)
│
└── README.md                           # This file
```

## Experimental Approaches

### 1. Baseline Single-Hypothesis Correction

Two prompting strategies with different risk profiles:

#### Cautious Strategy (`cautious.txt`)
- **Philosophy**: Prioritize precision over recall
- **Constraints**:
  - Text length within ±5% margins
  - Preserve natural speech patterns
  - Only unambiguous spelling errors and capitalization
- **Forbidden**: Restructuring, stylistic improvements, markdown
- **Models tested**: GPT-4.1, Kimi K2
- **Results**: Best overall performance (GPT-4.1: 19.2% WERR Estonian, 2.8% WERR Ukrainian)

#### Permissive Strategy (`permissive.txt`)
- **Philosophy**: Allow broader grammatical improvements
- **Approach**: Adapted from Fedorchenko & Alumäe (2025)
- **Constraints**: Estonian morphological rules, punctuation normalization
- **Risk**: Higher risk of overcorrection
- **Models tested**: GPT-4.1, Kimi K2
- **Results**: Consistently underperformed Cautious strategy

### 2. Multi-Agent Correction System

Three-agent system based on ReAct prompting with sequential processing:

#### Analysis Agent (`multi_agent/analysis_agent.txt`)
- **Role**: Error detection and hypothesis generation
- **Focus**: 4 error types (COMPOUND_ERROR, NER_ERROR, CODE_SWITCH_ERROR, MORPHOLOGICAL)
- **Tools**: PhoneticAnalyzerTool, ExaSearchTool, WebSearchTool
- **Language expertise**:
  - Estonian: 14 grammatical cases
  - Ukrainian: 7 grammatical cases

#### Search Agent (`multi_agent/search_agent.txt`)
- **Role**: Entity verification via web search
- **Tools**: Exa Search with language-specific domains
- **Verification**: Multiple source confirmation, confidence scoring
- **Domains**: et.wikipedia.org/eesti.ee (Estonian), uk.wikipedia.org/ukraine.ua (Ukrainian)

#### Synthesis Agent (`multi_agent/synthesis_agent.txt`)
- **Role**: Final correction decisions with self-validation
- **Principles**: Memory-first, linguistic validation, conservative approach
- **Preference**: Word editing over word addition/removal
- **Validation**: Three-level (phonetic, grounded, necessary)

**Results**: Only 2.3% WERR on Estonian, -1.2% WERR on Ukrainian (degraded performance)
- 16× more tokens than simple approaches
- 57× slower processing time
- Demonstrates that architectural complexity ≠ better performance

### 3. Reasoning Model with Constrained Cautious Strategy

#### Constrained Cautious (`constrained_cautious.txt`)
- **Model**: Gemini 2.5 Pro (reasoning model)
- **Base**: Cautious prompting strategy
- **Processing**: Batch processing (20 segments at once for better context)
- **Additional constraints**:
  - Explicit verbosity prevention
  - "LENGTH VIOLATION = AUTOMATIC REJECTION" for responses outside 70-130%
  - No explanations, reasoning trails, or markdown formatting
  - Optional Google Search grounding
  - Allows restructuring WITHIN segments for better flow
- **Optional enhancement**: Global context summary (`global_context_summary.txt`)
  - Generates <80 word summary of recording type, style, topics
  - Tested but showed no significant WER improvements
  - Can be enabled via `enable_global_summary` parameter
- **Languages**: Both Estonian and Ukrainian prompts available
- **Results**: Best Estonian performance (21.9% WERR), but degraded Ukrainian (-4.3% WERR)

## Key Findings from the Paper

### Performance Summary (Complete Results)

| Correction Model | Prompt | Search | Estonian WER%↓ | Ukrainian WER%↓ | Tokens/Word↓ | Words/Min↑ |
|------------------|--------|--------|----------------|-----------------|--------------|------------|
| **Baseline: Whisper large-v3-turbo** | - | - | 33.3 | 28.6 | - | - |
| **Non-reasoning models** | | | | | | |
| GPT-4.1 | Cautious | No | **26.9** | **27.9** | **7** | **2300** |
| GPT-4.1 | Cautious | Yes | 27.3 | **27.8** | 29 | 290 |
| GPT-4.1 | Permissive | No | 34.9 | 29.8 | **7** | **2300** |
| GPT-4.1 | Permissive | Yes | 28.1 | 30.9 | 29 | 290 |
| GPT-4.1 | Multi-Agent | Yes | 32.5 | 29.8 | 115 | 40 |
| Kimi K2 | Cautious | No | 29.7 | 28.3 | 8 | 790 |
| Kimi K2 | Cautious | Yes | 29.8 | 28.3 | 15 | 270 |
| Kimi K2 | Permissive | No | 29.8 | 28.9 | 8 | 790 |
| Kimi K2 | Permissive | Yes | 29.7 | 29.2 | 15 | 270 |
| **Reasoning models** | | | | | | |
| Gemini 2.5 Pro | Constrained Cautious | No | **26.0** | 29.8 | **7** | 55 |
| Gemini 2.5 Pro | Constrained Cautious | Yes | 26.1 | 29.7 | **7** | 55 |

**Key observations:**
- **Best Estonian**: Gemini 2.5 Pro Constrained Cautious (26.0% WER, 21.9% WERR)
- **Best Ukrainian**: GPT-4.1 Cautious (27.8-27.9% WER, 2.4-2.8% WERR)
- **Best efficiency**: GPT-4.1 Cautious without search (7 tokens/word, 2300 words/min)
- **Worst performance**: GPT-4.1 Permissive without search (34.9% Estonian - worse than baseline!)
- **Kimi K2**: Consistent but modest improvements (10.8% WERR Estonian, 1.0% WERR Ukrainian)

### Main Insights

1. **Simple prompting outperforms complexity**:
   - GPT-4.1 Cautious: 19.2% WERR (Estonian) with 7 tokens/word, 2300 words/min
   - Multi-agent: Only 2.3% WERR with 115 tokens/word (16× worse), 40 words/min (57× slower)
   - Architectural complexity = systematic performance degradation + massive computational cost

2. **Model-language compatibility matters**:
   - **Gemini 2.5 Pro**: Excellent on Estonian (21.9% WERR), but degrades Ukrainian (-4.3% WERR)
   - **GPT-4.1**: Consistent cross-linguistic performance (19.2% Estonian, 2.8% Ukrainian WERR)
   - **Kimi K2** (open-source): Modest improvements (10.8% Estonian, 1.0% Ukrainian WERR)
   - Model selection should consider target language characteristics

3. **Permissive strategies are risky**:
   - GPT-4.1 Permissive without search: 34.9% WER (worse than 33.3% baseline!)
   - Permissive with search: 28.1% WER (still worse than Cautious 26.9%)
   - Overcorrection is a real problem that degrades performance
   - Conservative constraints prevent this issue

4. **Search tools have minimal benefit**:
   - **Reasoning models**: 0.3% difference (Gemini 26.0% → 26.1%)
   - **Instruction-tuned**: Slight degradation (GPT-4.1 26.9% → 27.3% Estonian)
   - **Efficiency penalty**: 2-4× more tokens, 8-10× slower processing
   - **Kimi K2**: No benefit on Ukrainian (28.3% with/without search)
   - Native search integration (Gemini) doesn't improve results

5. **Architectural complexity fails systematically**:
   - Multi-agent with tools: 32.5% Estonian (2.3% WERR), 29.8% Ukrainian (-1.2% WERR)
   - **Context enhancements didn't help**: Systematic testing showed no improvements from:
     - Transcript summaries (`global_context_summary.txt`)
     - Surrounding segment context (3 previous + 3 next segments)
     - Named entity memory retention across sessions
   - Forced tool usage degraded performance even with theoretically useful information
   - Advanced models (GPT-4.1) failed to use tools spontaneously
   - Forcing reasoning onto instruction-tuned models = negative effects

6. **Overcorrection protection via prompting**:
   - No model fine-tuning required (unlike Udagawa et al. 2024)
   - No N-best list constraints needed (unlike Ma et al. 2023)
   - Simple prompt constraints provide effective protection
   - Cautious strategy prevents overcorrection across models and languages
   - More practical and resource-efficient than previous approaches

## Language-Specific Features

### Estonian
- **14 grammatical cases**: nominative, genitive, partitive, illative, inessive, elative, allative, adessive, ablative, translative, terminative, essive, abessive, comitative
- **Compound words**: Complex splitting/joining patterns
- **Code-switching**: Frequent English mixing
- **Dataset**: 12 podcast episodes, 10 hours, TalTech Estonian Speech Dataset 1.0

### Ukrainian
- **7 grammatical cases**: nominative, genitive, dative, accusative, instrumental, locative, vocative
- **Soft/hard consonants**: Palatalization effects
- **Stress patterns**: Mobile stress affecting vowel quality
- **Dataset**: 10 random recordings from AIDA Ukrainian Broadcast and Telephone Speech (LDC2023S01)

## Usage Notes

### For Estonian

**Best approach**: GPT-4.1 with Cautious strategy (26.9% WER, 19.2% WERR)
- Alternative: Gemini 2.5 Pro with Constrained Cautious (26.0% WER, 21.9% WERR) if slightly better accuracy is worth slower processing

**Prompt**: Use `estonian/cautious.txt` with GPT-4.1 or `estonian/constrained_cautious.txt` with Gemini 2.5 Pro

### For Ukrainian

**Best approach**: GPT-4.1 with Cautious strategy (27.9% WER, 2.8% WERR)
- Cross-linguistic robustness demonstrated
- Note: Gemini 2.5 Pro degraded performance on Ukrainian (-4.3% WERR)

**Prompts available**:
- `ukrainian/constrained_cautious.txt` for Gemini 2.5 Pro (not recommended based on results)
- Cautious strategy recommended (may need Ukrainian language adaptation)

### Processing
- Batch size: 20 segments
- Models: GPT-4.1 (`openai/gpt-4.1`), Gemini 2.5 Pro (`gemini-2.5-pro`), Kimi K2 (`moonshotai/kimi-k2-0905`)

## Citation

If you use these prompts in your research, please cite:

```bibtex
@unpublished{olev2025less,
  title={When Less is More: Simple Prompting Outperforms Complex Multi-Agent Systems for ASR Error Correction},
  author={Olev, Aivo and Alumäe, Tanel},
  note={Submitted to ICASSP 2026},
  year={2025}
}
```

## Related Work

- **Liberal correction baseline**: Fedorchenko & Alumäe (2025) - Optimizing LLM-Based Subtitle Correction for Estonian
- **ReAct prompting**: Yao et al. (2023) - ReAct: Synergizing Reasoning and Acting in Language Models
- **Overcorrection problem**: Udagawa et al. (2024) - Robust ASR Error Correction with Conservative Data Filtering
- **N-best correction**: Ma et al. (2023) - Can ChatGPT Correct ASR Errors?

## License

Prompts are released under MIT License. 