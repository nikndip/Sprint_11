# Go с нуля - 11 Спринт - Итоговое задание

## Sprint 12: GitHub Actions

Workflow `.github/workflows/ci.yml` выполняет `go vet` и `go test -race`
при push и Pull Request. При push тега после успешных проверок второй job
публикует образ в Docker Hub, скачивает его по digest и запускает приложение.

В Settings → Secrets and variables → Actions добавьте repository secrets:

- `DOCKERHUB_USERNAME` — ваш Docker ID.
- `DOCKERHUB_TOKEN` — Docker Hub access token с правом Read & Write.

Имя образа: `<Docker ID>/sprint11:<Git tag>`. Репозиторий Docker Hub должен
быть публичным, чтобы ревьювер мог скачать образ без авторизации.

После отправки workflow и настройки секретов создайте новый тег:

```bash
git tag v2
git push origin v2
```

Убедитесь, что оба jobs в запуске для тега завершились успешно, затем проверьте
скачивание и запуск (замените `nikndip` своим Docker ID, если он отличается):

```bash
docker pull nikndip/sprint11:v2
docker run --rm nikndip/sprint11:v2
```

Приложение выполняет демонстрационный сценарий и завершает работу; статус
контейнера `Exited (0)` ожидаем. На ревью отправляется ссылка на Pull Request.

Начальный код для выполнения итогового задания 11 спринта курса "Разработчик Go с нуля".

Информация для тех, кто не может выполнить финальное задание 12-го спринта из-за ограничения доступа к Docker Hub. Выход есть! Можно вместо реестра образов Docker Hub использовать GitHub Container Registry, который точно доступен в РФ.

В GitHub-экшне нужно будет внести несколько правок, чтобы все заработало. В первую очередь заводим вспомогательные переменные окружения с реестром ghcr.io (GitHub Container Resitry) и названием образа:

```yaml
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: my-app
```
Можно вместо конкретного названия образа использовать название репозитория:

```yaml
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
```

Далее в jobs с деплоем образа в реестр на шаге логина пишем:

```yaml
- uses: docker/login-action@v4
  with:
    registry: ${{ env.REGISTRY }} # Реестр GitHub Container Registry
    username: ${{ github.actor }} # подставится ваш логин GitHub
    password: ${{ secrets.GITHUB_TOKEN }} # токен авторизации GitHub
```

В таком случае не нужно заводить какие-то свои секреты secrets с данными из Docker Hub. Будем использовать стандартный токен GitHub. Затем на шаге мета-данных пишем:

```yaml
- uses: docker/metadata-action@v6
  with:
    images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }} # имя образа вида ghcr.io/name
```

Готово! С такими настройками экшна ваш образ опубликуется не в Docker Hub, а в реестре GitHub Container Registry. От чего, в принципе, суть не меняется. Но на всякий случай напишите еще комментарий в самом начале для ревьюера:

```yaml
# Использую GitHub Container Registry, т.к. Docker Hub недоступен
```


