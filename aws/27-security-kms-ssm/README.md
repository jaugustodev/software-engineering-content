# AWS Security & Encryption: KMS, Encryption SDK, SSM Parameter Store, IAM & STS

## Hands-on para assistir

| Aula | Por que assistir |
|------|-----------------|
| KMS Hands On w/ CLI | Ver criação de CMK, encrypt/decrypt via CLI — fundamental para entender KMS na prática |
| Encryption SDK CLI Hands On | Ver envelope encryption em ação via CLI |
| SSM Parameter Store Hands On (CLI) | Ver put-parameter, get-parameter com SecureString — muito usado no dia a dia |
| Secrets Manager - Hands On | Ver criação de secret e rotação automática no console |

## Teoria que posso explicar (pode pular as aulas)

- Encryption 101 (symmetric vs asymmetric, in-transit vs at-rest)
- KMS Overview (CMK, key policies, grants, key rotation)
- KMS Encryption Patterns and Envelope Encryption
- KMS Limits (throttling, DEK caching)
- KMS and AWS Lambda Practice
- S3 Bucket Key (reduz chamadas ao KMS)
- KMS Key Policies & IAM Principals
- CloudHSM Overview (hardware dedicado — nicho)
- SSM Parameter Store Overview (Standard vs Advanced, String vs SecureString)
- Secrets Manager - Overview (rotação automática, integração com RDS)
- SSM Parameter Store vs Secrets Manager (quando usar cada)
- CloudFormation - Secrets Manager & SSM Integration
- CloudWatch Logs Encryption
- CodeBuild Security
- AWS Nitro Enclaves (computação confidencial — conceito avançado)
