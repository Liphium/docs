# Frequently asked questions about setup

## Why do I need a domain for my town?

Alright, there are two reasons for this, let me explain.

First, if you don't have a domain, the server address of your town will be in all of the addresses of all of the people in your town. This would mean that if the address of the server ever changes (like if you want to move to a more powerful machine) everyone would lose their connections to people outside of your town. It would break everything in decentralization. The Liphium server is also designed in a way where it never expects to have the address of it change. Meaning if you don't have a domain and want to change the address of your server, you'll go through a lot of complaints from the **people in your town losing connections to friends in other towns**. And **you'd have to delete all of the accounts** as the server can't change the address of the friends you've saved in your vault. Would be kinda bad wouldn't it?

Second, by default Liphium blocks all towns that are using an insecure protocol. And because you can't have HTTPS when you just use a plain IP address you will never be able to connect to any towns running on the default configuration (which is basically all of them) because there are major security implications if you were to actually use a town without HTTPS in production.

I hope you understood the reasoning for this requirement and make the right choice. I will not help you with any of this stuff and if any request of this issue ever reaches me, I will ignore it. But if you have a good, secure and reasonable idea of how to solve this problem for everyone without huge migrations and want to implement all of it yourself, [feel free to file an issue](https://github.com/Liphium/station) as Liphium is open source. We can then discuss this further together.