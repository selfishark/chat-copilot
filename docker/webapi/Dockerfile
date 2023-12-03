# builder
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS builder

WORKDIR /source

# generate dev-certs for https
RUN dotnet dev-certs https

# copy everything else and build app
COPY webapi webapi
COPY shared shared

WORKDIR /source/webapi 

# Ensure NuGet packages are restored
RUN dotnet restore --use-current-runtime

# Install wget and add Azure.AI.FormRecognizer package
RUN apt update \
  && apt install -y wget \
  && dotnet add package Azure.AI.FormRecognizer --version 4.1.0

# Download language data for Tesseract
RUN wget -P /app/data https://github.com/tesseract-ocr/tessdata_best/raw/main/eng.traineddata \
  && wget -P /app/data https://github.com/tesseract-ocr/tessdata_best/raw/main/fra.traineddata

# Publish the app
RUN dotnet publish --use-current-runtime --self-contained false --no-restore -o /app

# Clean up apt-get
RUN apt-get clean \
  && rm -rf /var/lib/apt/lists/*

# final stage/image
FROM mcr.microsoft.com/dotnet/aspnet:7.0

ENV Kestrel__Endpoints__Http__Url=http://0.0.0.0:8080

WORKDIR /app

COPY --from=builder /app .
COPY --from=builder /root/.dotnet/corefx/cryptography/x509stores/my/* /root/.dotnet/corefx/cryptography/x509stores/my/

# Install required dependencies for Tesseract
RUN apt update \
  && apt install -y libleptonica-dev libtesseract-dev libc6-dev libjpeg62-turbo-dev libgdiplus \
  && ln -s /usr/lib/x86_64-linux-gnu/liblept.so.5 x64/libleptonica-1.82.0.so \
  && ln -s /usr/lib/x86_64-linux-gnu/libtesseract.so.4.0.1 x64/libtesseract50.so \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

ENTRYPOINT ["./CopilotChatWebApi"]
