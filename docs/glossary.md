# Glossary / Речник

> Keep the table in alphabetical order. One term per row. / Пазете таблицата по азбучен ред. Един термин на ред.

| Term / Термин | Meaning / Значение |
|---|---|
| CD | Continuous delivery: every merge to `main` produces a tested image in GHCR, ready to deploy. / Всяко сливане в `main` дава проверен образ в GHCR, готов за пускане. |
| CI | Continuous integration: every change is built and tested automatically. / Всяка промяна се изгражда и тества автоматично. |
| CVE | A publicly known security vulnerability with an ID; Trivy looks for them in the image. / Публично известна уязвимост с номер; Trivy ги търси в образа. |
| Deployment | Putting a version of the service where users can reach it. / Поставяне на версия на услугата там, където потребителите я достигат. |
| GHCR | GitHub Container Registry, where CI stores the images (`ghcr.io/ivokavalski/devops-labs-fn361223024`). / Регистърът, в който CI пази образите. |
| Health check | An endpoint that says whether the service is alive. / Адрес, който казва дали услугата е жива. |
| Issue | A task, bug or idea tracked on GitHub. / Задача, бъг или идея, проследявани в GitHub. |
| Pull request | A proposal to merge one branch into another, with review. / Предложение един клон да се слее в друг, с ревю. |
| Rollback | Running a previous, working image again by its tag (`APP_TAG=<sha>`). / Пускане отново на предишен работещ образ по таг. |
| Version | A named state of the service, e.g. 0.2.0. / Именувано състояние на услугата, напр. 0.2.0. |
