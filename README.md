Rest.jl
=============================================

[![Build Status](https://travis-ci.org/ylxdzsw/Rest.jl.svg?branch=master)](https://travis-ci.org/ylxdzsw/Rest.jl)

An easy way to build simple but extendable RESTful servers.

### Installation

```julia
Pkg.clone("git@github.com:ylxdzsw/Rest.jl.git","Rest")
```

### Example

##### Define a resource manually.

```julia
todolist = Resource("todolist")

addmethod(todolist, :GET) do req, _
    Response(200, JSON.json(collect(keys(_TODOLIST))))
end

addmethod(todolist, :POST) do req, _
    content       = JSON.parse(req[:body]|>ASCIIString)["content"]
    id            = string(hash(content))
    _TODOLIST[id] = content
    Response(200, JSON.json(Dict(:id=>id)))
end
```

##### Or using the @resource macro. (macros currently not work in julia v0.5)

```julia
@resource todoitem <: todolist begin
    :name  => "todoitem"
    :route => "*"

    "get a todoitem content"
    :GET => begin
        if haskey(_TODOLIST, id)
            Response(200, JSON.json(Dict(:content=>_TODOLIST[id])))
        else
            Response(404, JSON.json(Dict(:error=>"Not Found")))
        end
    end

    "add a todoitem with specific id"
    :PUT => begin
        content = JSON.parse(req[:body]|>ASCIIString)["content"]
        _TODOLIST[id] = content
        Response(200)
    end

    :DELETE => begin
        if haskey(_TODOLIST, id)
            delete!(_TODOLIST, id)
            Response(200)
        else
            Response(404, JSON.json(Dict(:error=>"Not Found")))
        end
    end
end
```

##### Run the server.

```julia
app = HttpHandler() do req::Request, res::Response
    todolist(req)
end

@async run(Server(app), host=ip"127.0.0.1", port=8000)
```

_notice that a Resource is a function that turns a Request to a Response, so it is a valid [Mux.jl](https://github.com/JuliaWeb/Mux.jl) endpoint._
