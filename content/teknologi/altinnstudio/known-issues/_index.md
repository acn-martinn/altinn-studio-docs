---
title: Known issues
description: Description of errors and weaknesses in Altinn Studio that developers should know about.
tags: [issues]
aliases:
 - /known-issues/
---

<div id="knownIssuesContent"></div>

<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
<script>
  let content = document.getElementById('knownIssuesContent');
  if (content.innerHTML === '') {
    content.innerHTML = 'Loading from github...';
  }  
   axios.get(`${'https://cors-anywhere.herokuapp.com/'}https://github.com/Altinn/altinn-studio/blob/master/KNOWNISSUES.md`)
    .then((res) => {
      const doc = new DOMParser().parseFromString(res.data, "text/html");

      content.innerHTML = doc.getElementById('readme').innerHTML;
      });
</script>
