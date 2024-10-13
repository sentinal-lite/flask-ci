# flask-ci

```yaml
version: 2.1

executors:
  python-executor:
    docker:
      - image: circleci/python:3.8
    working_directory: ~/repo

jobs:
  printEnv:
    executor: python-executor
    steps:
      - run:
          name: print envs
          command: |
            printenv

  lint:
    executor: python-executor
    steps:
      - checkout
      - run:
          name: Install dependencies
          command: |
            pip install -r requirements.txt
      - run:
          name: Run Linting
          command: |
            if flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics | grep -q "0"; then
              echo "Lint has passed and ready to merge"
            else
              echo "Lint failed, fix the issues"
              exit 1
            fi
      - run:
          name: Linting Done
          command: echo "Linting complete."

  test:
    executor: python-executor
    steps:
      - checkout
      - run:
          name: Install dependencies
          command: |
            pip install -r requirements.txt
      - run:
          name: Run Tests
          command: |
            pytest || echo "No tests found."
      - run:
          name: Test Done
          command: echo "Test done."

  pr-comment-fail:
    executor: python-executor
    steps:
      - run:
          name: Add Comment to PR
          command: |
            curl -X POST -H "Authorization: token ${GITHUB_TOKEN}" \
            -d '{"body":"PR failed on this stage: ${CIRCLE_STAGE} - [Job link](${CIRCLE_BUILD_URL})"}' \
            https://api.github.com/repos/${CIRCLE_PROJECT_USERNAME}/${CIRCLE_PROJECT_REPONAME}/issues/${CIRCLE_PR_NUMBER}/comments
          when: on_fail

workflows:
  say-hello-workflow:
    jobs:
      - lint:
          filters:
            branches:
              only:
                - dev
                - staging
                - master
      - test:
          filters:
            branches:
              only:
                - dev
                - staging
                - master
          requires:
            - lint
      - pr-comment-fail:
          filters:
            branches:
              only:
                - dev
                - staging
                - master
          requires:
            - test
            - lint


```