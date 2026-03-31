# Skill: @criar_projeto_modular

## Objetivo
Criar um projeto Java Spring já estruturado com arquitetura modular.

## Regras
- Deve aplicar automaticamente arquitetura modular
- Ignorar qualquer outra estrutura padrão
- Criar módulos por domínio

## Execução
1. Criar base do projeto (Spring, Lombok, MapStruct, Swagger)
2. Aplicar estrutura:

/modules
  /[feature]
    /controller
    /service
    /repository
    /mapper
    /domain

## Regra de ouro
Arquitetura modular sempre prevalece
