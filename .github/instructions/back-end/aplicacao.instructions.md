---
applyTo: "**/*AppServico.cs,**/*Profile.cs"
---

# Padrões da Camada Aplicação (<Projeto>.Aplicacao)

## Responsabilidade

Orquestrar fluxos de aplicação, mapear DTOs para comandos, delegar para serviços de domínio e retornar responses.

## Estrutura de Pastas

```
<Projeto>.Aplicacao/
└── <Feature>/
    ├── Profiles/
    │   └── <Feature>Profile.cs
    └── Servicos/
        ├── Interfaces/
        │   └── I<Feature>AppServico.cs
        └── <Feature>AppServico.cs
```

## Nomenclatura

| Elemento  | Padrão                 | Exemplo                  |
| --------- | ---------------------- | ------------------------ |
| Serviço   | `<Feature>AppServico`  | `DepoimentosAppServico`  |
| Interface | `I<Feature>AppServico` | `IDepoimentosAppServico` |
| Profile   | `<Feature>Profile`     | `DepoimentosProfile`     |

## Padrões de Código

### Interface

```csharp
namespace <Projeto>.Aplicacao.<Feature>.Servicos.Interfaces;

public interface I<Feature>AppServico
{
    Task<<Feature>Response> InserirAsync(<Feature>InserirRequest request);
    Task<<Feature>Response> EditarAsync(<Feature>EditarRequest request);
    Task ExcluirAsync(int id);
    <Feature>Response Recuperar(int id);
    PaginacaoConsulta<<Feature>Response> Listar(<Feature>ListarRequest request);
}
```

### Serviço de Aplicação com UnitOfWork

```csharp
namespace <Projeto>.Aplicacao.<Feature>.Servicos;

public class <Feature>AppServico : I<Feature>AppServico
{
    private readonly I<Feature>Servicos _<feature>Servicos;
    private readonly IUnitOfWork _unitOfWork;
    private readonly IMapper _mapper;

    public <Feature>AppServico(
        I<Feature>Servicos <feature>Servicos,
        IUnitOfWork unitOfWork,
        IMapper mapper)
    {
        _<feature>Servicos = <feature>Servicos;
        _unitOfWork = unitOfWork;
        _mapper = mapper;
    }

    public async Task<<Feature>Response> InserirAsync(<Feature>InserirRequest request)
    {
        try
        {
            await _unitOfWork.BeginTransactionAsync();
            var comando = _mapper.Map<<Feature>InserirComando>(request);
            var entidade = await _<feature>Servicos.InserirAsync(comando);
            await _unitOfWork.CommitAsync();
            return _mapper.Map<<Feature>Response>(entidade);
        }
        catch
        {
            await _unitOfWork.RollbackAsync();
            throw;
        }
    }

    public async Task<<Feature>Response> EditarAsync(<Feature>EditarRequest request)
    {
        try
        {
            await _unitOfWork.BeginTransactionAsync();
            var comando = _mapper.Map<<Feature>EditarComando>(request);
            var entidade = await _<feature>Servicos.EditarAsync(comando);
            await _unitOfWork.CommitAsync();
            return _mapper.Map<<Feature>Response>(entidade);
        }
        catch
        {
            await _unitOfWork.RollbackAsync();
            throw;
        }
    }

    public async Task ExcluirAsync(int id)
    {
        try
        {
            await _unitOfWork.BeginTransactionAsync();
            await _<feature>Servicos.ExcluirAsync(id);
            await _unitOfWork.CommitAsync();
        }
        catch
        {
            await _unitOfWork.RollbackAsync();
            throw;
        }
    }

    public <Feature>Response Recuperar(int id)
    {
        var entidade = _<feature>Servicos.Recuperar(id);
        return _mapper.Map<<Feature>Response>(entidade);
    }

    public PaginacaoConsulta<<Feature>Response> Listar(<Feature>ListarRequest request)
    {
        var resultado = _<feature>Servicos.Listar(request);
        return new PaginacaoConsulta<<Feature>Response>
        {
            Registros = _mapper.Map<List<<Feature>Response>>(resultado.Registros),
            Total = resultado.Total
        };
    }
}
```

### Profile AutoMapper

```csharp
namespace <Projeto>.Aplicacao.<Feature>.Profiles;

public class <Feature>Profile : Profile
{
    public <Feature>Profile()
    {
        CreateMap<<Feature>InserirRequest, <Feature>InserirComando>();
        CreateMap<<Feature>EditarRequest, <Feature>EditarComando>();
        CreateMap<<Entidade>, <Feature>Response>();
    }
}
```

## Regras

- ✅ UnitOfWork em toda operação de escrita (Inserir, Editar, Excluir)
- ✅ Mapear DTOs para comandos via AutoMapper
- ✅ Delegar lógica para serviços de domínio
- ❌ NÃO conter lógica de negócio
- ❌ NÃO acessar repositórios diretamente
- ❌ NÃO usar UnitOfWork em consultas (Listar, Recuperar)

## Registro (IoC)

```csharp
services.AddScoped<I<Feature>AppServico, <Feature>AppServico>();
// Profile em ConfiguracoesAutoMapper:
config.AddProfile<<Feature>Profile>();
```
