# CI/CD Integrations

## GitHub Actions

```yaml
name: Deploy Live Update

on:
  workflow_dispatch:
    inputs:
      channel:
        description: "Channel to deploy to"
        required: true
        default: "default"

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run build
      - run: npx @capawesome/cli login --token ${{ secrets.CAPAWESOME_CLOUD_TOKEN }}
      - run: |
          npx @capawesome/cli apps:channels:create \
            --name ${{ github.event.inputs.channel }} \
            --ignore-errors
      - run: |
          npx @capawesome/cli apps:liveupdates:upload \
            --channel ${{ github.event.inputs.channel }} \
            --path dist \
            --yes
```

Store `CAPAWESOME_CLOUD_TOKEN` as a repository secret.

## GitLab CI/CD

```yaml
deploy-live-update:
  image: node:20
  stage: deploy
  rules:
    - when: manual
  variables:
    CHANNEL:
      value: "default"
      description: "Channel to deploy to"
  script:
    - npm ci
    - npm run build
    - npx @capawesome/cli login --token $CAPAWESOME_CLOUD_TOKEN
    - npx @capawesome/cli apps:channels:create --name $CHANNEL --ignore-errors
    - npx @capawesome/cli apps:liveupdates:upload --channel $CHANNEL --path dist --yes
```

Store `CAPAWESOME_CLOUD_TOKEN` as a masked CI/CD variable.

## Azure DevOps

```yaml
trigger: none

pool:
  vmImage: "ubuntu-latest"

parameters:
  - name: channel
    displayName: "Channel"
    type: string
    default: "default"

steps:
  - task: NodeTool@0
    inputs:
      versionSpec: "20.x"
  - script: npm ci
  - script: npm run build
  - script: npx @capawesome/cli login --token $(CAPAWESOME_CLOUD_TOKEN)
  - script: npx @capawesome/cli apps:channels:create --name ${{ parameters.channel }} --ignore-errors
  - script: npx @capawesome/cli apps:liveupdates:upload --channel ${{ parameters.channel }} --path dist --yes
```

Store `CAPAWESOME_CLOUD_TOKEN` as a secure pipeline variable.

## Bitbucket Pipelines

```yaml
pipelines:
  custom:
    deploy-live-update:
      - variables:
          - name: CHANNEL
            default: "default"
      - step:
          name: Deploy Live Update
          image: node:20
          script:
            - npm ci
            - npm run build
            - npx @capawesome/cli login --token $CAPAWESOME_CLOUD_TOKEN
            - npx @capawesome/cli apps:channels:create --name $CHANNEL --ignore-errors
            - npx @capawesome/cli apps:liveupdates:upload --channel $CHANNEL --path dist --yes
```

Store `CAPAWESOME_CLOUD_TOKEN` as a repository variable.
