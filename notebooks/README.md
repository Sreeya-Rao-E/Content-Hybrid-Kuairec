## Experimental Design Rationale

### Why hold embedding dimension (64) and optimizer fixed across Feature-On/Feature-Off

DeepFM and Wide & Deep already have a fixed slot for categorical content features
sitting alongside the ID embeddings. Toggling content on/off changes only what
populates that slot — not the network topology, the FM interaction order, or the
wide component's cross-feature logic. Holding embedding dimension and optimizer
(learning rate, batch size, regularization) identical across both runs means the
ID-embedding capacity and optimization dynamics are the same in both conditions.
Any performance delta between Feature-On and Feature-Off is then attributable to
the *presence of content signal itself* — not to a difference in model capacity
(a bigger embedding table can memorize more regardless of content) or a
difference in optimization trajectory (a different learning rate can reach a
different optimum independent of the data it's fed). Without this control, a
performance gap could be explained by architecture/capacity differences rather
than content, which would invalidate the isolation the experiment is designed to achieve.

### Why comparing *effect size* across MovieLens and KuaiRec is confounded

Direction (does content help, hurt, or do nothing, within a fixed architecture)
is a clean within-model comparison and travels across datasets. Magnitude does not,
because the two datasets' content vocabularies differ in ways that are entangled
with, but not identical to, "how useful is content":

- **Different vocabulary size and granularity.** MovieLens genres are an 18-way,
  coarse, decades-curated taxonomy. KuaiRec's video-category tags are a 14-way
  taxonomy built for a structurally different content type (short-form video).
  With embedding dimension fixed, the same dimension allocates different effective
  representational capacity depending on how many categories exist and how they're
  distributed — "same embedding size" does not mean "same expressiveness" across
  the two feature spaces.
- **Different informativeness per feature is entangled with dataset difficulty.**
  A genre tag on a well-studied, decades-old ratings dataset and a category tag on
  a short-video platform aren't drawn from comparable information sources. An
  observed size difference could reflect a genuine difference in how useful content
  is, or it could just reflect one taxonomy being a noisier encoding of similar
  underlying usefulness — the design as built can't cleanly separate these.
- **Different headroom above the CF baselines.** Catalog size, interaction density,
  and the train/eval protocol (KuaiRec's sparse-train/dense-eval split vs.
  MovieLens's standard split) put the pure-CF (LightGCN/SVD) ceiling at a different
  point on each dataset. A larger or smaller content lift can be a ceiling/floor
  artifact of how much room is left to move the needle, independent of the
  feature's intrinsic value.

**Conclusion:** this design is well-powered to say *whether* content helps, hurts,
or is neutral on each dataset, and to compare those signs across datasets. A claim
like "content helps 2x more on MovieLens than KuaiRec" would conflate feature
vocabulary richness with genuine content-value differences and is not supported by
this design without further controls (e.g., normalizing vocabulary size or running
an ablation that matches category granularity).

## Predicted Outcomes (expected_findings)

1. **DeepFM, MovieLens (on vs. off):** Predicted moderate positive lift — genre
   embeddings should interact through the FM second-order term and pick up real
   signal. **Null:** no meaningful difference, which would itself be notable given
   how consistently this effect is reported in the original DeepFM literature.

2. **Wide & Deep, MovieLens (on vs. off):** Predicted a smaller positive lift than
   DeepFM's, since Wide & Deep's benefit depends on hand-specified wide-component
   crosses that this design doesn't build for genres. **Null:** indistinguishable
   performance, plausible if the wide component's memorization dominates regardless
   of content.

3. **DeepFM, KuaiRec (on vs. off):** Predicted null or small negative effect —
   coarse category tags plus the fully-observed eval protocol (which removes the
   exposure-bias shortcut content features sometimes exploit) should leave ID
   embeddings doing most of the work. **Null hypothesis for this comparison (and
   the core hypothesis of the project):** if the content lift is statistically
   indistinguishable in both direction and size from the MovieLens lift, that would
   mean the content-hybrid literature generalizes to KuaiRec's short-video,
   fully-observed setting just as well as it does to MovieLens's sparse,
   genre-tagged setting — undercutting the paper's motivating premise that this
   dataset's protocol and content structure would behave differently.

4. **Wide & Deep, KuaiRec (on vs. off):** Predicted null, for the same reasons as
   (3), possibly slightly negative if extra category embeddings mostly add
   optimization noise at this vocabulary's granularity.

5. **LightGCN vs. SVD (both datasets):** Predicted LightGCN > SVD on MovieLens by
   the typically reported margin; predicted the gap narrows on KuaiRec since
   LightGCN's sparsity-mitigation advantage matters less when the eval matrix is
   already near-fully-observed. **Null:** no narrowing — LightGCN retains the same
   relative advantage regardless of protocol.

6. **Content-off hybrids vs. pure-CF baselines, KuaiRec:** Predicted Feature-Off
   DeepFM/Wide&Deep perform roughly on par with LightGCN/SVD (graceful degradation
   to CF-equivalent when content doesn't help). **Null:** Feature-Off hybrids
   underperform both CF baselines, suggesting a fixed architectural cost
   independent of content that would complicate the isolation logic above.
