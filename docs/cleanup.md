---
id: cleanup
title: "Clean Up"
previous_page: exercise4
---
<link rel="stylesheet" href="assets/css/styles.css">

# Clean Up
When you're done testing you can delete all the kind clusters you no longer care aboout.
1. Show all kind clusters.
   ```bash
   kind get clusters
   ```
1. Delete a kind cluster.
   ```bash
   kind delete cluster --name ${cluster_name}
   ```

<br />
