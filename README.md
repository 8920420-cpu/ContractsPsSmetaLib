contracts-lib/
│
├── proto/
│   ├── base/
│   │   └── v1/
│   │       ├── context.proto
│   │       ├── status.proto
│   │       └── error_codes.proto
│   │
│   ├── person/
│   │   └── v1/
│   │       ├── person.proto
│   │       └── person_service.proto
│   │
│   ├── account/
│   │   └── v1/
│   │       ├── account.proto
│   │       └── account_service.proto
│   │
│   ├── common/
│   │   └── v1/
│   │       └── pagination.proto   # на будущее
│   │
│   └── README.md
│
├── buf.yaml            # (рекомендуется)
├── buf.gen.yaml        # генерация SDK
├── LICENSE
└── README.md



protoc -I=proto `
  --go_out=. --go_opt=paths=source_relative `
  --go-grpc_out=. --go-grpc_opt=paths=source_relative `
  proto/findate/v1/findate.proto

  -I — include path (папка, откуда protoc будет искать все импорты внутри .proto)

  --go_out=.
    Генерирует Go-код для сообщений (message)
.   → сгенерированные файлы положит в текущую папку (или рядом с исходным .proto)

  ---go-grpc_out=.
     Генерирует Go-код для gRPC (service), если в proto есть rpc-методы
    . → генерировать в текущей папке

 --proto/findate/v1/sendfindate.proto
    Это сам файл, который нужно скомпилировать
    protoc читает этот файл и все его импорты (с учётом -I)

    
# Для каждого файла вручную
protoc -I=proto --go_out=gen/go --go_opt=paths=source_relative --go-grpc_out=gen/go --go-grpc_opt=paths=source_relative proto/findate/v1/findate.proto
protoc -I=proto --go_out=gen/go --go_opt=paths=source_relative --go-grpc_out=gen/go --go-grpc_opt=paths=source_relative proto/base/v1/context.proto
protoc -I=proto --go_out=gen/go --go_opt=paths=source_relative --go-grpc_out=gen/go --go-grpc_opt=paths=source_relative proto/base/v1/status.proto
protoc -I=proto --go_out=gen/go --go_opt=paths=source_relative --go-grpc_out=gen/go --go-grpc_opt=paths=source_relative proto/base/v1/error_codes.proto


Get-ChildItem -Recurse -Filter *.proto | ForEach-Object {
    protoc -I=proto --go_out=gen/go --go_opt=paths=source_relative --go-grpc_out=gen/go --go-grpc_opt=paths=source_relative $_.FullName
}

# Папка с proto файлами
$protoRoot = "proto"

# Папка для сгенерированных Go файлов
$outDir = "gen/go"

# Создаём папку gen/go, если её нет
if (-not (Test-Path $outDir)) {
    New-Item -ItemType Directory -Path $outDir -Force
}

# Рекурсивно перебираем все .proto файлы и генерируем
Get-ChildItem -Recurse -Filter *.proto $protoRoot | ForEach-Object {
    Write-Host "Generating $($_.FullName)"
    protoc -I=$protoRoot `
           --go_out=$outDir --go_opt=paths=source_relative `
           --go-grpc_out=$outDir --go-grpc_opt=paths=source_relative `
           $_.FullName
}