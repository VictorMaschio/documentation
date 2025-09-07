


```bash
docker run --rm -it \
  -v "$(pwd)/backend:/home/app" \
  -w /home/app \
  node:24.4-alpine \
  sh -c "npm install -g @nestjs/cli && \
         nest new . --skip-git --package-manager=npm --strict --skip-install && \
         npm install && \
         npm install @prisma/client -y && \
         npx prisma init && \
         npm install --save-dev prisma typedoc typedoc-plugin-markdown"
```