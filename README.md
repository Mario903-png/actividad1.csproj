
using QuintoProyecto.Models;
using QuintoProyecto.Services;
using Spectre.Console;

var clienteService = new ClienteService();
var ordenService = new OrdenService();

while (true)
{
    AnsiConsole.Clear();

    var opcion = AnsiConsole.Prompt(
        new SelectionPrompt<string>()
            .Title("[cyan]SISTEMA DE GESTIÓN DE LAVANDERÍA[/]")
            .PageSize(10)
            .AddChoices(
                "1. Registrar cliente",
                "2. Registrar orden de lavandería",
                "3. Mostrar órdenes",
                "4. Salir"
            )
    );

    switch (opcion)
    {
        case "1. Registrar cliente":
        {
            AnsiConsole.Clear();
            AnsiConsole.MarkupLine("[yellow]REGISTRAR CLIENTE[/]");

            var nombre = AnsiConsole.Ask<string>("Nombre del cliente:");
            var telefono = AnsiConsole.Ask<string>("Teléfono del cliente:");

            if (string.IsNullOrWhiteSpace(nombre) ||
                string.IsNullOrWhiteSpace(telefono))
            {
                AnsiConsole.MarkupLine("[red]El nombre y el teléfono son obligatorios.[/]");
            }
            else
            {
                clienteService.Registrar(nombre, telefono);
                AnsiConsole.MarkupLine("[green]Cliente registrado correctamente.[/]");
            }

            AnsiConsole.MarkupLine("\nPresiona una tecla para continuar.");
            Console.ReadKey(true);
            break;
        }

        case "2. Registrar orden de lavandería":
        {
            AnsiConsole.Clear();
            AnsiConsole.MarkupLine("[yellow]REGISTRAR ORDEN DE LAVANDERÍA[/]");

            var clientes = clienteService.ObtenerTodos();

            if (clientes.Count == 0)
            {
                AnsiConsole.MarkupLine("[red]Primero debes registrar un cliente.[/]");
            }
            else
            {
                var nombresClientes = clientes
                    .Select(c => $"{c.Id}. {c.Nombre}")
                    .ToList();

                var clienteSeleccionado = AnsiConsole.Prompt(
                    new SelectionPrompt<string>()
                        .Title("Selecciona el cliente:")
                        .AddChoices(nombresClientes)
                );

                var clienteId = int.Parse(clienteSeleccionado.Split('.')[0]);
                var cliente = clientes.First(c => c.Id == clienteId);

                var prendas = AnsiConsole.Ask<string>(
                    "Describe las prendas (ej. 3 camisas y 2 pantalones):");

                var precio = AnsiConsole.Ask<decimal>(
                    "Precio total de la orden (RD$):");

                if (string.IsNullOrWhiteSpace(prendas) || precio < 0)
                {
                    AnsiConsole.MarkupLine("[red]Las prendas son obligatorias y el precio no puede ser negativo.[/]");
                }
                else
                {
                    ordenService.Registrar(cliente.Nombre, prendas, precio);
                    AnsiConsole.MarkupLine("[green]Orden registrada correctamente.[/]");
                }
            }

            AnsiConsole.MarkupLine("\nPresiona una tecla para continuar.");
            Console.ReadKey(true);
            break;
        }

        case "3. Mostrar órdenes":
        {
            AnsiConsole.Clear();
            var ordenes = ordenService.ObtenerTodas();

            if (ordenes.Count == 0)
            {
                AnsiConsole.MarkupLine("[yellow]Todavía no hay órdenes registradas.[/]");
            }
            else
            {
                var tabla = new Table()
                    .Title("Órdenes de lavandería")
                    .BorderColor(Color.Cyan)
                    .RoundedBorder();

                tabla.AddColumn("ID");
                tabla.AddColumn("Cliente");
                tabla.AddColumn("Prendas");
                tabla.AddColumn("Precio (RD$)");
                tabla.AddColumn("Estado");

                foreach (var orden in ordenes)
                {
                    tabla.AddRow(
                        orden.Id.ToString(),
                        orden.NombreCliente,
                        orden.Prendas,
                        orden.Precio.ToString("N2"),
                        orden.Estado
                    );
                }

                AnsiConsole.Write(tabla);
            }

            AnsiConsole.MarkupLine("\nPresiona una tecla para continuar.");
            Console.ReadKey(true);
            break;
        }

        case "4. Salir":
            AnsiConsole.MarkupLine("[green]Aplicación finalizada. ¡Hasta luego![/]");
            return;
    }
}
