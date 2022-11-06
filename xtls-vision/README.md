## XTLS Vision, TLS in TLS, to the stars and beyond

Originally posted by [**@yuhan6665**](https://github.com/yuhan6665) at [Xray-core discussions #1295](https://github.com/XTLS/Xray-core/discussions/1295) and translated by Google Translate.

## 1. XTLS in simple language

The core logic of XTLS is to use real TLS to hide the proxy traffic from the most common traffic on the Internet.

For ordinary TLS proxy protocols (such as the original Trojan), the user's proxy client establishes a real TLS connection with the proxy server, and through this encrypted tunnel, the application (such as a browser) establishes a TLS connection with the target server (such as Google) connect. At this time, the browser and the Google server are end-to-end encrypted, and no one (including the proxy service) can decrypt or disguise the information sent. This is the so-called e2e or end-to-end encryption.

```
browser ---- proxy client ==== proxy server ---- Google
```

When the proxy data passes through the censor, it is actually encrypted twice, and RPRX is keenly aware that 99% of the traffic does not need additional encryption during this process. Because TLS 1.3 encrypted data is identical in appearance, censors cannot tell the difference. That is to say, the proxy logic only needs:

1. Establish a real TLS link
2. Inside the encrypted tunnel, track the communication between the browser and Google, identifying whether the current communication is TLS 1.3
3. If not, use normal TLS proxy method and continue to use encrypted tunnel
4. If so, wait for the two ends to start transmitting encrypted data, and the proxy client inserts the UUID in the first inner layer encrypted packet, instructing the proxy server: we are ready to run naked!
5. After the second inner encrypted packet, XTLS does not do anything with the data packet, but simply copies

This is `xtls-rprx-vision`.

We can find out that:

* Proxy-initiated outer TLS is real, so censors cannot break it.
* The streaking traffic is simply copied in terms of proxy. If the censors do any damage, they will get a normal alert response from the browser and Google server, which is no different from ordinary access.
* The signal of streaking is UUID, this signal is agent private information to avoid similar. [About the exploit of the code feature in the 23 3 3 judgment part](https://github.com/XTLS/Go/issues/17) Go#17 vulnerability.

## 2. Almost perfect

On October 3, we started to identify and block TLS proxies on a large scale. Through some related discussions and insider information, we know that the censors used a lot of resources and machine learning methods. As we all know, the specialty of machine learning is to extract statistical features from massive data.

To quote [**@klzgrad**](https://github.com/klzgrad/naiveproxy#known-weaknesses), an important weakness of normal TLS proxies is the nesting doll. As mentioned in the above discussion, although the appearance of the encrypted package is indistinguishable to the censors, the inevitable point of the encrypted nesting doll is that a data packet header will be added to each packet. The more encryption layers, the heavier the packet header will be.

Although this increment is not large, it may be obvious for small data (such as reply packets), and there will be some packet lengths that exceed the MTU limit of the bottom layer of the network. On top of that, since each packet adds the same length, it may have certain statistical characteristics.

The main reason why RPRX invented XTLS was to reduce additional encryption. We are now launching a new flow control Vision because XTLS has a unique ability to fight censors. It can be said that when transmitting TLS 1.3 data, XTLS 99% of the packets have almost perfect traffic characteristics. Because it is raw data, it has not been processed by any proxy.

## 3. TLS in TLS

Why 99% of the packets are raw data? What exactly is the 1%?

We need to continue to explore what a typical proxy is doing when it encounters internal TLS 1.3 traffic in the first few packets.

Visually, when the encrypted channel is established:

```
The first package proxy client -> proxy server "Hello, the target address of this proxy visit is Google, here is my UUID"
The second package proxy client <- proxy server "Hello, received your proxy request, please start sending data"
The third package Browser -> Proxy Client -> Proxy Server -> Google "Hello Google, I will make an encrypted call with you. I support encryption methods." (This package is also called TLS Client Hello)
The fourth package browser <- proxy client <- proxy server <- Google "Hello user this is the Google certificate this time will be encrypted with TLS_AES_128_GCM_SHA256 Let's start encrypting calls!" (this package is also called TLS Server Hello)
Fifth package Browser -> Proxy Client -> Proxy Server -> Google "Received Let's start encrypting calls!"
```

The proxy protocol you know is the same, as long as the user uses any TLS connection, the above-mentioned handshake process is required. As mentioned earlier, the outer TLS encryption can be considered absolutely secure, but for the examiner, in addition to the cracking information, some additional information can also be used to identify these 5 packets.

This is the crux of the so-called TLS in TLS detection.

## 4. Life and death 5 packets

The most obvious feature is the length of these 5 packets.

```
The first packet is very short and the only variable is the destination address
The second packet is very short and almost fixed
Third packet short very few changes almost the only variable is the target SNI
The length of the fourth packet varies greatly
The fifth packet is very short with few changes
```

It can be intuitively felt that their packet length characteristics are actually very obvious. In Vision, our response is also very simple, which is to fill the length of each short packet to an interval of 900 to 1400. Note that this method is different from traditional random padding. We do not blindly add padding to all packets, but based on our analysis of internal traffic, we target and pad the iconic packets of the TLS handshake process.

Another relatively hidden feature is called timing feature. Limited by the design of the TLS handshake, you can notice that the order of the first few packets is fixed. That is to say, after the browser sends a TLS Client Hello, it must wait for Google to send a TLS Server Hello, and after that the browser must send a "Received Let's start encrypting the call!", so that the subsequent conversation can be normal. conduct.

If the user sends a packet to the server, it is called C, and the opposite direction is called S. Can the reviewer determine that this is a TLS in TLS connection based on the timing of data such as CSCSC, and their time interval characteristics?

We believe that it is not enough to draw conclusions, and Vision has not dealt with this aspect.

There are many developers mentioning MUX multiplexing against TLS in TLS detection. It does have an obfuscating effect in this regard. If two different TLS connections are multiplexed in a tunnel, there is a chance to form various timing characteristics such as CCSSCC.

It can be expected that the over-the-wall technology war will continue to upgrade from shadowsocks (encryption) -> active detection -> TLS -> machine learning, and whether it will survive or die will largely depend on the processing of these five packets.

## 5. Frequently Asked Questions

Q: The padding length of Vision is hard-coded, does it also have statistical characteristics?

A: First of all, there is no absolute security in the concept of network security. In theory, any encryption can be cracked by brute force, but it is only the amount of calculation (invested resources). As designers of security protocols, we only need to increase the difficulty of cracking and identification to a level beyond the reach of reviewers. How high is this height? I'm afraid we need to see the war to find out. 

Going back to what we already know, the length distribution of TLS 1.2 and 1.3 handshakes is very stable. Several key packages to access mainstream websites (e.g. Google) can be seen as fixed length. If the random length of Vision's existing large packets is compared with the handshake packets without padding, the recognition difficulty should increase by an order of magnitude.

Also, in some of the leaked discussions, it was mentioned that machine learning can identify random fills below 40%. This can prove that machine learning has an upper limit of recognition.

Q: Why is the new flow control named Vision?

A: We think Vision represents the original design philosophy of the RPRX. The first version of XTLS was actually developed during exploration, and was limited by the development environment at that time, and some parts were too complicated. It can be said that Vision represents the ideal form and future direction of XTLS. RPRX says:

> Flow.. What problem is it trying to solve? It affects macro-traffic timing characteristics, not micro-characteristics, which is what encryption is meant to address. The traffic timing characteristics can be brought by the protocol, such as the SOCKS5 handshake when SOCKS5 over TLS. Different characteristics on TLS are different protocols for the monitor. At this time, unlimited Schedulers is equivalent to an unlimited protocol (redistribute each time The amount of data sent, etc.)

[v2ray/v2ray-core#2636](https://github.com/v2ray/v2ray-core/issues/2636)

> In fact, Direct can be implemented without magic modification of the TLS library. It does not require read filtering, or even write filtering when transmitting TLSv1.3. It is very simple and efficient.

[XTLS/Go#12 (comment)](https://github.com/XTLS/Go/issues/12#issuecomment-770205658)

Q: The new flow control is so good, do you recommend everyone to use it to bypass the wall?

A: As other developers have said, for current reviewers, it is very dangerous to put all your eggs in one basket. So we encourage everyone to make good use of the features of each tool and to spread the features.

* [**@klzgrad**](https://github.com/klzgrad) [NaiveProxy](https://github.com/klzgrad/naiveproxy) uses Chrome browser architecture as a proxy client and Caddy as a proxy server, which can highly imitate a normal HTTPS access, it also includes TLS handshake and other feature packet filling
* [**@gfw-report**](https://github.com/gfw-report) just spent an afternoon to write [magic Shadowsocks](https://github.com/net4people/bbs/issues/136) by injecting additional data to change the characteristics of traditional shadowsocks disordered traffic
* In the case of pure self-use, using MITM proxy, currently there are few tools in this way, you can refer to [Advanced usage of V2Ray (2): MITM](https://github.com/KiriKira/Kirikira.moe/blob/master/articles-md/V2Ray%E7%9A%84%E8%BF%9B%E9%98%B6%E6%95%99%E7%A8%8B(2):MITM.md)

Q: Do you recommend national secrets such as [SM4](https://en.wikipedia.org/wiki/SM4_(cipher)) for the future?

A: Not recommended, as mentioned at the beginning, we think the safest way is to hide in the most common traffic on the Internet. In addition to HTTPS, we should consider what kind of outbound traffic is common. SSH? Remote Desktop? WeChat video? Email (fog)? Radio communication (fog)? Instead of using an extremely niche encryption and protocol.

## 6. Our point of view

In fact, TLS-based proxies have been quite mature in 2017. Since the Shadowsocks service was blocked on a large scale in 2019, it is conceivable that the censors have invested a lot of manpower, material and computing power, and it took at least three years to have today's technology. As of now, we can see that there are still some false closures, most of the CDNs are still alive, and the IPV6 is still alive. Judging from the number of visits to the Xray project and the number of online telegram groups hitting new highs, the effect of the censors in blocking information this time, Actually very limited.

As a member of the development circle, it is no exaggeration to say that our "broken iron" volunteer team, Shadowsocks, *ray, Trojan, Naive, Hystaria, Tuic, Sing, has been pressing the censors in the friction on the ground is still fancy.

The TLS proxy war has just begun. The reason why the censors have to use AI means just proves from the side that the previous censorship methods, including active detection, are ineffective for TLS proxies. In the face of AI opponents, we must do better at the detail level.
We can give full play to our strengths, brainstorming, open source, decentralized features. Specifically, as a proxy tool and service, it can directly filter inner traffic, which is actually a natural advantage for us to censors. Using this information gap, Vision's logic can pad the critical TLS handshake. This idea is consistent with [NaiveProxy's padding for some specific packets](https://github.com/klzgrad/naiveproxy#proxy-payload-padding).

Thank you for your patience in reading this article. We welcome the majority of technology enthusiasts to leave a message to communicate with us. Every message and idea will become a force in our battle.

Lastly to [**@RPRX**](https://github.com/RPRX)

The green hills do not change, the green waters flow forever, may you return as a teenager.
