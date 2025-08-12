### Derivations of the (Soft) EM Algorithm for Document Clustering

Here are the derivations for the Expectation and Maximization steps of the (soft) EM algorithm for document clustering, including the model parameters and their update expressions.

#### **Model Parameters**

In the context of document clustering, we aim to learn the following parameters:
*   **Mixing Coefficients (Cluster Priors) $\phi$**: These represent the prior probability of a document belonging to a specific cluster. For K clusters, we have a vector $\phi = (\phi_1, \phi_2, ..., \phi_K)$, where $\phi_k \geq 0$ and $\sum_{k=1}^{K} \phi_k = 1$.
*   **Word Probabilities per Cluster $\mu$**: This is a set of parameters $\mu = (\mu_1, \mu_2, ..., \mu_K)$, where each $\mu_k$ is a vector representing the probability distribution of words in the vocabulary for cluster *k*. So, $\mu_k = (\mu_{k,1}, \mu_{k,2}, ..., \mu_{k, |A|})$, where *|A|* is the size of the vocabulary, $\mu_{k,w} \geq 0$, and $\sum_{w \in A} \mu_{k,w} = 1$.

#### **Expectation (E) Step**

The E-step calculates the posterior probability, or "responsibility," of each cluster *k* for each document *d*. This tells us how likely it is that a given document belongs to each cluster, based on the current parameter estimates.

The responsibility $\gamma(z_{nk})$ which is the posterior probability $P(z_{nk}=1 | d_n, \theta^{old})$ is calculated as follows:

$\gamma(z_{nk}) = \frac{\phi_k \prod_{w \in d_n} (\mu_{k,w})^{c(w, d_n)}}{\sum_{j=1}^{K} \phi_j \prod_{w \in d_n} (\mu_{j,w})^{c(w, d_n)}}$

Where:
*   $\gamma(z_{nk})$ is the responsibility of cluster *k* for document *n*.
*   $c(w, d_n)$ is the count of word *w* in document $d_n$.

This step essentially computes the expected values for the latent variables, which in this case are the cluster assignments.

#### **Maximization (M) Step**

The M-step updates the model parameters ($\phi$ and $\mu$) to maximize the expected log-likelihood of the observed data, given the responsibilities calculated in the E-step.

The updated parameters are calculated as follows:

**Update for mixing coefficients $\phi_k$:**

The new mixing coefficient for each cluster *k* is the average responsibility of that cluster over all documents.

$\phi_k^{new} = \frac{\sum_{n=1}^{N} \gamma(z_{nk})}{N}$

Where *N* is the total number of documents.

**Update for word probabilities per cluster $\mu_{k,w}$:**

The new probability of a word *w* in a cluster *k* is the weighted average of the count of that word in all documents, where the weight is the responsibility of that cluster for the document.

$\mu_{k,w}^{new} = \frac{\sum_{n=1}^{N} \gamma(z_{nk}) c(w, d_n)}{\sum_{v=1}^{|A|} \sum_{n=1}^{N} \gamma(z_{nk}) c(v, d_n)}$

This step finds the maximum likelihood estimates for the parameters based on the "completed" data from the E-step.

The EM algorithm alternates between the E-step and M-step until the parameters converge.
