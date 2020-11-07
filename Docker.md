# Docker 安全最佳实践

## 不要在环境变量里存储秘钥

使用 `ENV` 去保存秘钥是一个很糟糕的尝试。因为 Dockerfiles 经常跟随应用分发，所以这其实跟硬编码秘钥在代码里没什么区别。

如何检测:

```dockerfile
secrets_env = [
  "passwd",
  "password",
  "pass",
  "pwd", # can't use this one   
  "secret",
  "key",
  "access",
  "api_key",
  "apikey",
  "token",
  "tkn"
]

deny[msg] {    
  input[i].Cmd == "env"
  val := input[i].Value
  contains(lower(val[_]), secrets_env[_])
  msg = sprintf("Line %d: Potential secret in ENV key found: %s", [i, val])
}
```

## 只使用那些信任的基础镜像 

如何检测：

```dockerfile
deny[msg] {
  input[i].Cmd == "from"
  val := split(input[i].Value[0], "/")
  count(val) > 1
  msg = sprintf("Line %d: use a trusted base image", [i])
}
```

## 不要在基础镜像上使用 `latest` tag

如何检测：

```dockerfile
deny[msg] {
  input[i].Cmd == "from"
  val := split(input[i].Value[0], ":")
  contains(lower(val[1]), "latest"])
  msg = sprintf("Line %d: do not use 'latest' tag for base images", [i])
}
```

## 避免使用 curl 脚本

如何检测：

```dockerfile
deny[msg] {
  input[i].Cmd == "run"
  val := concat(" ", input[i].Value)
  matches := regex.find_n("(curl|wget)[^|^>]*[|>]", lower(val), -1)
  count(matches) > 0
  msg = sprintf("Line %d: Avoid curl bashing", [i])
}
```

## 不要升级你的系统包

如何检测：

```dockerfile
upgrade_commands = [
  "apk upgrade",
  "apt-get upgrade",
  "dist-upgrade",
]

deny[msg] {
  input[i].Cmd == "run"
  val := concat(" ", input[i].Value)
  contains(val, upgrade_commands[_])
  msg = sprintf(“Line: %d: Do not upgrade your system packages", [i])
}
```

## 尽可能的不要使用 `ADD`

如何检测：

```dockerfile
deny[msg] {
  input[i].Cmd == "add"
  msg = sprintf("Line %d: Use COPY instead of ADD", [i])
}
```

## 不要 root

如何检测：

```dockerfile
any_user {
  input[i].Cmd == "user"
}

deny[msg] {
  not any_user
  msg = "Do not run as root, use USER instead"
}
```

## 不要 sudo

如何检测：

```dockerfile
deny[msg] {
  input[i].Cmd == "run"
  val := concat(" ", input[i].Value)
  contains(lower(val), "sudo")
  msg = sprintf("Line %d: Do not use 'sudo' command", [i])
}
```