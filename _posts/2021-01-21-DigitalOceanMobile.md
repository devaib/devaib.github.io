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

    var url = location.href;
    var redictedURL = url.replace("https://devaib.github.io/digitaloceanmobile", "digitaloceanmobile:authenticate");
    console.log(redictedURL);

    window.location.replace(redictedURL);
}
window.onload = redirectToDigitalOceanMobile;
</script>