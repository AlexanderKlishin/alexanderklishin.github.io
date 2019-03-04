---
layout: post
categories: oracle oci
---

[Examples on how to code the client to use timeouts][src]

{% highlight c %}
    int fd;

    /* get fd */
    OCISvcCtxToLda(svchp, errhp, &lda);
    ognfd(&lda,&fd);
    OCILdaToSvcCtx(&svchp, errhp, &lda);

    /* close on timer */
    close(fd);
{% endhighlight %}

[src]: https://twiki.cern.ch/twiki/pub/PSSGroup/OCIClientHangProtection/OCI_code_with_timeouts.txt

