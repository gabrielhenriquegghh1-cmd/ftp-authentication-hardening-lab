# Laboratório de autenticação e proteção do FTP

[Read in English](README.md) · [Guia de publicação](PUBLISHING.md#português)

Este projeto de cibersegurança da DIO analisa como um serviço FTP legado responde a logins aceitos e rejeitados e quais evidências ficam no servidor. A prática usou autenticação manual. Medusa, força bruta e password spraying são abordados como conceitos do curso.

**Resultado.** A VM Metasploitable 2 respondeu aos três pings. O cliente FTP mostrou um login aceito e um rejeitado, e o log do servidor mostrou falhas de autenticação. A porta TCP 21 continuou aberta na última checagem, então o efeito de parar o FTP ainda precisa ser verificado.

## Ambiente

| Componente | Configuração observada |
| --- | --- |
| Virtualização | Oracle VirtualBox |
| Servidor | Metasploitable 2 em `192.168.56.101/24` |
| Rede | Adaptador 1 do servidor no modo Host-only, em `vboxnet0` |
| Cliente | Terminal Linux pela interface `vboxnet0` do hospedeiro, em `192.168.56.1/24` |
| Serviço FTP | O banner do servidor informou `vsFTPd 2.3.4` na porta TCP 21 |
| Data da evidência | 17 de novembro de 2026, conforme o log FTP do servidor |

As capturas do cliente mostram a interface do VirtualBox no hospedeiro. Portanto, a evidência documenta testes do hospedeiro para a máquina virtual pela rede Host-only.

## Método e evidências

No cliente Linux, a checagem usou o IPv4 da VM exibido pelo `ifconfig`:

```bash
LAB_IP=192.168.56.101
ping -c 3 "$LAB_IP"
nc -vz -w 3 "$LAB_IP" 21
```

O cliente FTP foi aberto de forma interativa com `ftp "$LAB_IP"`. As duas sessões observadas usaram nomes de usuário diferentes. A sessão aceita usou a conta dedicada `ftp-lab`. Isso limita a comparação porque o nome de usuário também mudou entre os dois resultados.

| Verificação | Observação | Evidência |
| --- | --- | --- |
| Rede virtual | Adaptador 1 conectado à rede Host-only `vboxnet0` | [Configuração do VirtualBox](images/01-host-only-network.png) |
| Endereço do servidor | `eth0` exibiu `192.168.56.101/24` | [Interface do servidor](images/02-server-ipv4.png) |
| Teste inicial | 3 de 3 respostas ao ping, 0% de perda e TCP 21 aberta | [Checagem de conectividade](images/03-ftp-connectivity.png) |
| Autenticação manual | A sessão rejeitada retornou `530 Login incorrect`. A conta dedicada retornou `230 Login successful`. | [Respostas do cliente](images/04-ftp-login-outcomes.png) |
| Registros do servidor | O trecho do log FTP mostrou dois eventos `FAIL LOGIN` de `192.168.56.1`, às 15:11:47 e às 15:13:07 no horário do servidor. | [Eventos de falha do FTP](images/05-ftp-failure-logs.png) |
| Reteste | 3 de 3 respostas ao ping, 0% de perda e TCP 21 ainda aberta | [Reteste de conectividade](images/06-ftp-connectivity-recheck.png) |

O trecho consultado de `/var/log/auth.log` não mostrou eventos FTP dessas sessões. O log específico do serviço mostrou as falhas, mas o trecho disponível não mostrou o evento de sucesso correspondente à conta `ftp-lab`. A resposta do cliente comprova o login aceito. As capturas do cliente não exibem horários, o que limita a correlação individual com os eventos do servidor.

As capturas foram recortadas e tiveram dados ocultados para publicação. Elas mantêm os endereços do laboratório e os resultados, com o nome de usuário pessoal, o nome da máquina e os detalhes de interfaces alheias ao teste ocultos.

## Conceitos do curso

O Medusa é uma ferramenta que automatiza tentativas de autenticação. A força bruta pode gerar uma série de falhas em uma conta. O password spraying usa poucas senhas candidatas em muitas contas e pode produzir falhas distribuídas entre elas. Na investigação, compare as contas, as origens e os horários. A prática deste projeto usou logins manuais e não atribuiu os eventos a uma ferramenta ou técnica de ataque.

## Mitigação e validação

A medida proposta é parar o FTP se ele for desnecessário. No servidor, identifique o processo que escuta na porta TCP 21 e pare o serviço pelo gerenciador correspondente. Registre o estado da porta no servidor após a alteração e repita os testes de ping e TCP no mesmo cliente. Uma VM que continua respondendo, sem processo escutando na porta e com a conexão TCP recusada, é compatível com a interrupção do serviço. A persistência após reinicialização exige uma checagem separada.

Em ambientes que ainda precisam transferir arquivos, use SFTP quando apropriado, restrinja as origens de rede autorizadas e monitore falhas de autenticação. Falhas repetidas em uma conta ou distribuídas por várias contas ajudam a orientar a investigação. Esses sinais, sozinhos, não identificam a ferramenta ou a técnica empregada.

## Referências

- [Manual do Oracle VirtualBox · Redes Host-only](https://docs.oracle.com/en/virtualization/virtualbox/7.2/user/networkingdetails.html#host-only-networking)
- [RFC 959 · Protocolo FTP e códigos de resposta](https://www.rfc-editor.org/rfc/rfc959.html)
- [OWASP Authentication Cheat Sheet · Limitação de tentativas e monitoramento](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
