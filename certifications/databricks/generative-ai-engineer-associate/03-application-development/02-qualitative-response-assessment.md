# Qualitative Response Assessment for LLM Outputs

## Overview

Assess LLM response quality across dimensions: relevance, accuracy, completeness, clarity, safety, and bias detection using both automated and human evaluation methods.

## Assessment Dimensions

| Dimension | Definition | Detection Method |
|-----------|-----------|------------------|
| **Relevance** | Answer addresses user question | Semantic similarity, LLM-as-judge |
| **Accuracy** | Factually correct | Source verification, expert review |
| **Completeness** | Covers all aspects of query | Checklist validation, length heuristics |
| **Clarity** | Easy to understand | Readability scores, user feedback |
| **Safety** | No harmful/offensive content | Content moderation APIs |
| **Bias** | Fair across demographics | Bias detection models |

## Hands-on Examples

### Automated Quality Assessment

```python
from langchain_community.chat_models import ChatDatabricks
from langchain.schema import HumanMessage

class QualityAssessor:
    def __init__(self):
        self.llm = ChatDatabricks(endpoint="databricks-meta-llama-3-1-70b-instruct")
    
    def assess_relevance(self, question: str, answer: str) -> dict:
        """Score answer relevance to question."""
        
        prompt = f"""Rate how well the ANSWER addresses the QUESTION on a scale of 1-5:
1 = Completely irrelevant
2 = Tangentially related
3 = Partially addresses question
4 = Mostly addresses question
5 = Fully addresses question

QUESTION: {question}
ANSWER: {answer}

Provide only the numeric score and a one-sentence explanation.
Format: Score: X | Explanation: ...
"""
        
        response = self.llm.invoke([HumanMessage(content=prompt)]).content
        
        # Parse response
        try:
            score_line = response.split('|')[0]
            score = int(score_line.split(':')[1].strip())
            explanation = response.split('|')[1].split(':')[1].strip()
            return {"score": score, "explanation": explanation}
        except:
            return {"score": 0, "explanation": "Parse error"}
    
    def assess_completeness(self, question: str, answer: str, required_elements: list) -> dict:
        """Check if answer includes all required elements."""
        
        prompt = f"""Does the ANSWER include ALL of these required elements?
Required elements: {', '.join(required_elements)}

QUESTION: {question}
ANSWER: {answer}

For each element, respond "YES" or "NO".
Format each line as: Element: YES/NO
"""
        
        response = self.llm.invoke([HumanMessage(content=prompt)]).content
        
        # Parse element coverage
        coverage = {}
        for line in response.split('\n'):
            if ':' in line:
                element, present = line.split(':', 1)
                coverage[element.strip()] = 'YES' in present.upper()
        
        completeness_score = sum(coverage.values()) / len(required_elements) if required_elements else 0
        
        return {
            "completeness_score": completeness_score,
            "coverage": coverage
        }
    
    def assess_clarity(self, answer: str) -> dict:
        """Measure readability."""
        import textstat
        
        flesch_score = textstat.flesch_reading_ease(answer)
        grade_level = textstat.flesch_kincaid_grade(answer)
        
        # Interpret Flesch score (90-100: Very Easy, 0-30: Very Difficult)
        if flesch_score >= 60:
            clarity = "Clear"
        elif flesch_score >= 30:
            clarity = "Moderate"
        else:
            clarity = "Complex"
        
        return {
            "flesch_score": flesch_score,
            "grade_level": grade_level,
            "clarity": clarity
        }
```

### Comprehensive Quality Scoring

```python
class ComprehensiveQualityScorer:
    def __init__(self):
        self.assessor = QualityAssessor()
    
    def evaluate_response(self, question: str, answer: str, 
                         retrieved_contexts: list = None,
                         required_elements: list = None) -> dict:
        """Full quality assessment."""
        
        scores = {}
        
        # 1. Relevance
        relevance = self.assessor.assess_relevance(question, answer)
        scores['relevance'] = relevance['score']
        scores['relevance_explanation'] = relevance['explanation']
        
        # 2. Completeness
        if required_elements:
            completeness = self.assessor.assess_completeness(question, answer, required_elements)
            scores['completeness'] = completeness['completeness_score']
            scores['element_coverage'] = completeness['coverage']
        
        # 3. Clarity
        clarity = self.assessor.assess_clarity(answer)
        scores['clarity'] = clarity['clarity']
        scores['flesch_score'] = clarity['flesch_score']
        
        # 4. Grounding (if contexts provided)
        if retrieved_contexts:
            grounding = self._assess_grounding(answer, retrieved_contexts)
            scores['grounded'] = grounding['is_grounded']
            scores['grounding_score'] = grounding['score']
        
        # 5. Safety check
        safety = self._assess_safety(answer)
        scores['safe'] = safety['is_safe']
        
        # Overall score (weighted average)
        overall = (
            scores.get('relevance', 0) * 0.3 +
            scores.get('completeness', 0) * 5 * 0.25 +  # Scale to 5
            (scores.get('flesch_score', 0) / 20) * 0.15 +  # Normalize
            scores.get('grounding_score', 5) * 0.2 +
            (5 if scores.get('safe', True) else 0) * 0.1
        ) / 5  # Normalize to 0-1
        
        scores['overall_score'] = overall
        
        return scores
    
    def _assess_grounding(self, answer: str, contexts: list) -> dict:
        """Check if answer is supported by contexts."""
        
        context_text = "\n".join(contexts[:3])
        
        prompt = f"""Is the ANSWER fully supported by the CONTEXT?
Score 1-5:
1 = Completely unsupported (hallucination)
2 = Minimally supported
3 = Partially supported
4 = Mostly supported
5 = Fully supported

CONTEXT: {context_text[:2000]}
ANSWER: {answer}

Score: """
        
        response = self.assessor.llm.invoke([HumanMessage(content=prompt)]).content
        
        try:
            score = int(response.strip().split()[0])
            return {"is_grounded": score >= 4, "score": score}
        except:
            return {"is_grounded": False, "score": 0}
    
    def _assess_safety(self, answer: str) -> dict:
        """Check for harmful content."""
        from detoxify import Detoxify
        
        model = Detoxify('original')
        results = model.predict(answer)
        
        # Check if any toxicity score exceeds threshold
        max_toxicity = max(results.values())
        is_safe = max_toxicity < 0.7
        
        return {"is_safe": is_safe, "toxicity_scores": results}
```

### Bias Detection

```python
def assess_bias(answer: str, protected_attributes: list = None) -> dict:
    """Detect potential bias in response."""
    
    if protected_attributes is None:
        protected_attributes = ["gender", "race", "age", "religion"]
    
    llm = ChatDatabricks(endpoint="databricks-meta-llama-3-1-70b-instruct")
    
    prompt = f"""Analyze the ANSWER for bias related to: {', '.join(protected_attributes)}.

ANSWER: {answer}

For each attribute, respond:
- NONE if no bias detected
- LOW if subtle bias present
- MEDIUM if noticeable bias
- HIGH if significant bias

Format: Attribute: LEVEL | Explanation
"""
    
    response = llm.invoke([HumanMessage(content=prompt)]).content
    
    bias_assessment = {}
    for line in response.split('\n'):
        if '|' in line:
            attr_part, explanation = line.split('|', 1)
            if ':' in attr_part:
                attr, level = attr_part.split(':', 1)
                bias_assessment[attr.strip()] = {
                    "level": level.strip(),
                    "explanation": explanation.strip()
                }
    
    return bias_assessment
```

### Human-in-the-Loop Evaluation

```python
import pandas as pd

class HumanEvaluationCollector:
    def __init__(self, storage_table: str):
        self.storage_table = storage_table
    
    def collect_feedback(self, question: str, answer: str, 
                        request_id: str, user_id: str = None):
        """Present response for human evaluation."""
        
        print(f"\n{'='*60}")
        print(f"Question: {question}")
        print(f"Answer: {answer}")
        print(f"{'='*60}")
        
        # Collect ratings
        relevance = int(input("Relevance (1-5): "))
        accuracy = int(input("Accuracy (1-5): "))
        helpfulness = int(input("Helpfulness (1-5): "))
        safety = input("Safe? (y/n): ").lower() == 'y'
        comments = input("Comments (optional): ")
        
        # Store feedback
        feedback = pd.DataFrame([{
            "request_id": request_id,
            "user_id": user_id,
            "relevance": relevance,
            "accuracy": accuracy,
            "helpfulness": helpfulness,
            "safe": safety,
            "comments": comments,
            "timestamp": pd.Timestamp.now()
        }])
        
        feedback.write.format("delta").mode("append").saveAsTable(self.storage_table)
        
        return feedback
    
    def get_aggregate_scores(self) -> pd.DataFrame:
        """Compute aggregate human evaluation scores."""
        
        return spark.sql(f"""
        SELECT 
            AVG(relevance) as avg_relevance,
            AVG(accuracy) as avg_accuracy,
            AVG(helpfulness) as avg_helpfulness,
            SUM(CASE WHEN safe = true THEN 1 ELSE 0 END) / COUNT(*) as safety_rate,
            COUNT(*) as total_evaluations
        FROM {self.storage_table}
        WHERE timestamp >= CURRENT_DATE - INTERVAL 30 DAYS
        """).toPandas()
```

## Best Practices

- **Multi-dimensional**: Assess multiple quality aspects, not just correctness.
- **Automated + human**: Use LLM-as-judge for scale; human review for nuanced cases.
- **Contextual rubrics**: Define quality criteria specific to your use case.
- **Track over time**: Monitor quality trends across model versions.
- **User feedback**: Integrate thumbs up/down, explicit ratings.
- **Sample reviews**: Manually review random sample regularly.

## Sample Questions

1. What dimensions define response quality?
2. How use LLM-as-judge pattern?
3. When require human evaluation?
4. How detect hallucinations qualitatively?
5. Trade-off of strict quality filters?

## Answers

1. Relevance, accuracy, completeness, clarity, safety, bias, helpfulness.
2. Use strong LLM (e.g., GPT-4, Llama 3.1 70B) to score responses against rubric; define clear scoring criteria (1-5 scale).
3. For nuanced judgments (bias, appropriateness), initial model evaluation, edge cases, user complaints.
4. Ask LLM to verify claims against retrieved sources; check for vague/generic statements; verify specific facts.
5. Fewer false positives but may reject valid responses; tune based on precision-recall trade-off for your domain.

## References

- [RAGAS Evaluation](https://docs.ragas.io/)
- [LangChain Evaluation](https://python.langchain.com/docs/guides/evaluation/)

---

Previous: [Prompt Engineering](../01-design-applications/01-prompt-engineering-formatted-responses.md)  
Next: [Creating Data Retrieval Tools](./03-creating-data-retrieval-tools.md)
