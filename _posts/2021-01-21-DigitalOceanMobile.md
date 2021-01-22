---
layout: post
title: Digital Ocean Mobile
description: Digital Ocean Management Tool on iOS.
summary: Digital Ocean Management Tool on iOS.
tags: [digitaloceanmobile]
permalink: "/digitaloceanmobile"
---

A Digital Ocean Management app on iOS using API v2.

<script type="text/javascript">
function redirectToDigitalOceanMobile() {
    var location = document.location;
    console.log(location);


    var url = "https://devaib.github.io/digitaloceanmobile:authenticate?code=fc91c9a1f3e238ef7ce6aab6146d5743f26cfaedafa51c1a98bef0b3c85ea0c0";
    var redictedURL = url.replace("https://devaib.github.io/digitaloceanmobile:authenticate", "digitaloceanmobile:authenticate");
    console.log(redictedURL);

    window.location.replace(redictedURL);
}
window.onload = redirectToDigitalOceanMobile;
</script>