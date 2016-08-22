Occasionally, I develop my spark jobs on the remote cluster which causes some trouble when I try to push my commits to github. 
My private key is not stored on the remote machine, so Github rejects my pushes. 
You can fix this by forwarding your credentials through ssh.
Super nifty:

```
ssh -A <address>
ssh git@github.com
>  Hi harterrt! You've successfully authenticated, but GitHub does not provide shell access.
```

If you're having trouble, [this page](https://developer.github.com/guides/using-ssh-agent-forwarding/) has debugging ideas.
In particular, make sure `ssh-add -L` lists your public key.
