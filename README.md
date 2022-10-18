# GitHub Action - Waiter

If you are waiting for EC2 instance to change its state, please DO NOT use this action.

You can do something like this instead

```yml
- name: Wait for EC2 instance become running state
  run: aws ec2 wait instance-running --instance-ids ${{ steps.other-step-x.outputs.EC2_ID }}
```

## Example:

```yml
- name: Wait for Runner become Available and "online" or "offline" in GitHub
  uses: acerorg/gha-waiter@v1
  with:
    delay: 15
    max_attempts: 30
    expected_result: online  # or "offline"
    checker_cmd: |
      curl -s -u "git:${{ secrets.TOKEN }}" \
              -H "Accept: application/vnd.github.v3+json" \
              ${{ github.api_url }}/repos/${{ github.repository }}/actions/runners \
      | jq -r ".runners[] | select(.name | contains(\"${{ matrix.runner_name }}\")) | .status"
```

```yml
- name: Wait for spot instance requests to become "active" or "disabled" in AWS
  uses: acerorg/gha-waiter@v1
  env:
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  with:
    delay: 5
    max_attempts: 30
    expected_result: active  # or "disabled"
    checker_cmd: |
      aws ec2 describe-spot-instance-requests \
              --spot-instance-request-ids ${{ steps.other-step-x.outputs.REQUEST_ID }} \
      | jq -r '.SpotInstanceRequests[0].State'
```

For aws logs, there is a quota limit on the AWS account, and you will see `ThrottlingException` easily.

```yml
- name: Wait for CloudWatch Logs Stream become available
  uses: acerorg/gha-waiter@v1
  env:
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  with:
    delay: 15
    max_attempt: 10
    expected_result: 1
    checker_cmd: |
      aws logs describe-log-streams \
            --log-group-name '/aws/ec2/GitHub-Runner/${{ env.GH_REPO_NAME }}' \
            --log-stream-name-prefix '${{ steps.other-step-x.outputs.EC2_ID }}/${{ matrix.ip }}/cloud-init-output.log' \
      | jq -r '.logStreams | length'
```

```yml
- name: Monitoring CloudWatch Log Stream
  uses: acerorg/gha-waiter@v1
  env:
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  with:
    delay: 15
    max_attempt: 10
    expected_result: 1
    checker_cmd: |
      aws logs tail '/aws/ec2/GitHub-Runner/${{ env.GH_REPO_NAME }}' \
                --log-stream-names '${{ steps.other-step-x.outputs.EC2_ID }}/${{ matrix.ip }}/cloud-init-output.log' \
      | grep -c ' Started GitHub Actions Runner '
```
