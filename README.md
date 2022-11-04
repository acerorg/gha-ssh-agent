# GitHub Action - SSH agent

## Example:

```yml
- name: Start ssh agent
  uses: acerorg/gha-ssh-agent/start@v0
  with:
    ssh-key: ${{ secrets.SSH_PRIVATE_KEY }}
```

```yml
- name: Kill ssh agent
  if: always()
  uses: acerorg/gha-ssh-agent/stop@v0
```
