# CLI Development

## Profile

### What It Is
CLI (Command Line Interface) development in Python covers building user-friendly command-line tools with argument parsing, subcommands, interactive prompts, and rich terminal output. Modern Python CLIs use `typer` (type-hint-driven) or `click` (decorator-based), with `rich` for beautiful terminal formatting.

### Origin and Evolution
- `argparse` (2006, stdlib) — Standard library argument parser
- `click` (2014, Armin Ronacher) — Decorator-based CLI framework
- `rich` (2020, Will McGuigan) — Terminal formatting, tables, progress bars
- `typer` (2020, Sebastian Ramirez) — Type-hint-driven CLI (built on click)
- `textual` (2022, Will McGuigan) — Terminal UI (TUI) framework
- Current: typer + rich is the modern stack for CLI development

### Key Authors and References
- **Armin Ronacher** — click creator (also Flask)
- **Sebastian Ramirez** — typer creator (also FastAPI)
- **Will McGuigan** — rich and textual creator
- **Python documentation** — argparse module

### Key Concepts at a Glance
| Tool | Approach |
|------|----------|
| typer | Type hints → CLI arguments. `def main(name: str, count: int = 1)` |
| click | Decorators: `@click.command()`, `@click.option()`, `@click.argument()` |
| argparse | Imperative: `parser.add_argument("--name")` |
| rich | Beautiful output: tables, progress bars, syntax highlighting, panels |
| textual | Full TUI applications with widgets (interactive terminal UI) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Type hints as CLI definition** — With typer, function parameters become CLI arguments. `name: str` → required argument. `count: int = 1` → optional with default. No boilerplate.
2. **Help text for everything** — Every command, argument, and option should have help text. Users shouldn't need to read source code to use the tool.
3. **Rich output for humans** — Use rich for tables, progress bars, and colored output. Machine output (piped) should be plain text or JSON.
4. **Exit codes matter** — `sys.exit(0)` for success, `sys.exit(1)` for error. Other tools depend on these.
5. **Subcommands for complex CLIs** — Group related commands: `app users list`, `app users create`. Don't put everything in one flat command.

### When to Use vs. When to Avoid

**typer**: Modern CLIs, type-hint-driven development, when you want minimal boilerplate
**click**: Complex CLIs with custom parameter types, established ecosystem
**argparse**: Simple scripts, no external dependencies, stdlib-only environments
**rich**: Any CLI that outputs to a terminal (tables, progress, formatted text)

---

## Frameworks and Methodologies

### 1. CLI Application Design — step-by-step
1. Define the command structure (single command or subcommands)
2. Choose framework: typer (recommended), click, or argparse
3. Define commands with parameters and help text
4. Add output formatting with rich
5. Handle errors with clear messages and proper exit codes
6. Add shell completion support
7. Package as installable script entry point

### 2. Interactive CLI — step-by-step
1. Use `rich.prompt.Prompt` or `typer.prompt()` for user input
2. Add `rich.progress` for long operations
3. Use `rich.table` for tabular output
4. Add `--json` flag for machine-readable output
5. Support piping (detect if stdout is a TTY)

---

## How to Apply

### Decision Checklist
- [ ] Are all commands and options documented with help text?
- [ ] Is output formatted for humans (rich) with a --json option?
- [ ] Are exit codes correct (0 success, 1 error)?
- [ ] Are errors displayed clearly with actionable messages?
- [ ] Is the CLI installable as a script entry point?

### Implementation Patterns

**Typer Application:**
```python
import typer
from rich.console import Console
from rich.table import Table
from typing import Annotated, Optional

app = typer.Typer(help="Order management CLI")
console = Console()

@app.command()
def list_orders(
    customer: Annotated[Optional[str], typer.Option(help="Filter by customer ID")] = None,
    status: Annotated[str, typer.Option(help="Filter by status")] = "all",
    limit: Annotated[int, typer.Option(help="Max results")] = 20,
    json_output: Annotated[bool, typer.Option("--json", help="JSON output")] = False,
):
    """List orders with optional filters."""
    orders = fetch_orders(customer=customer, status=status, limit=limit)

    if json_output:
        import json
        typer.echo(json.dumps([o.dict() for o in orders], indent=2))
        return

    table = Table(title="Orders")
    table.add_column("ID", style="cyan")
    table.add_column("Customer")
    table.add_column("Status", style="green")
    table.add_column("Total", justify="right")

    for order in orders:
        table.add_row(order.id, order.customer_id, order.status, f"${order.total:.2f}")

    console.print(table)

@app.command()
def create(
    customer_id: Annotated[str, typer.Argument(help="Customer ID")],
    items: Annotated[list[str], typer.Option("--item", help="Product IDs")],
    dry_run: Annotated[bool, typer.Option(help="Preview without creating")] = False,
):
    """Create a new order."""
    if dry_run:
        console.print("[yellow]DRY RUN — order not created[/yellow]")
        console.print(f"Customer: {customer_id}, Items: {items}")
        return

    order = create_order(customer_id, items)
    console.print(f"[green]Order {order.id} created successfully![/green]")

if __name__ == "__main__":
    app()
```

**Subcommands with Typer:**
```python
import typer

app = typer.Typer()
users_app = typer.Typer(help="User management")
orders_app = typer.Typer(help="Order management")

app.add_typer(users_app, name="users")
app.add_typer(orders_app, name="orders")

@users_app.command("list")
def list_users():
    """List all users."""
    ...

@users_app.command("create")
def create_user(name: str, email: str):
    """Create a new user."""
    ...

@orders_app.command("list")
def list_orders():
    """List all orders."""
    ...

# CLI: app users list, app users create "Alice" "a@b.com"
# CLI: app orders list
```

**Rich Output Patterns:**
```python
from rich.console import Console
from rich.progress import track, Progress
from rich.panel import Panel
from rich.syntax import Syntax

console = Console()

# Progress bar for iterations
for item in track(items, description="Processing..."):
    process(item)

# Custom progress with multiple tasks
with Progress() as progress:
    download = progress.add_task("Downloading...", total=100)
    process = progress.add_task("Processing...", total=100)

    for i in range(100):
        progress.update(download, advance=1)
    for i in range(100):
        progress.update(process, advance=1)

# Panels for important information
console.print(Panel(
    "[bold green]Deployment successful![/bold green]\n"
    f"Version: {version}\n"
    f"Environment: {env}",
    title="Status",
))

# Syntax highlighting
code = 'print("Hello, World!")'
console.print(Syntax(code, "python", theme="monokai"))
```

**Error Handling:**
```python
import sys
import typer
from rich.console import Console

console = Console(stderr=True)  # Errors to stderr

@app.command()
def deploy(environment: str):
    """Deploy to the specified environment."""
    try:
        result = run_deployment(environment)
        console.print(f"[green]Deployed to {environment}[/green]")
    except PermissionError:
        console.print(f"[red]Error: No permission to deploy to {environment}[/red]")
        raise typer.Exit(code=1)
    except ConnectionError as e:
        console.print(f"[red]Error: Cannot connect to {environment}: {e}[/red]")
        raise typer.Exit(code=1)

# Entry point in pyproject.toml:
# [project.scripts]
# my-cli = "my_app.cli:app"
```

### Common Mistakes
1. **No help text** — Commands and options without descriptions; users can't discover features
2. **Print instead of rich** — Plain `print()` for everything; use rich for structured output
3. **Wrong exit codes** — Returning 0 on error; other tools rely on exit codes for error detection
4. **No --json flag** — CLI output that can't be parsed by other tools; support machine-readable output
5. **Monolithic commands** — One command with 20 flags instead of subcommands

### Integration with Other Topics
- **Web Frameworks** — CLI management commands alongside web apps
- **Testing** — `typer.testing.CliRunner` for CLI testing
- **Configuration** — CLI reads from Pydantic BaseSettings
- **Project Structure** — CLI entry point in `[project.scripts]`
- **Error Handling** — CLI translates exceptions to user-friendly error messages

---

## Resources

### Essential Reading
- typer documentation (typer.tiangolo.com)
- click documentation (click.palletsprojects.com)
- rich documentation (rich.readthedocs.io)

### Online Resources
- Real Python — CLI tutorials
- textual documentation (textual.textualize.io)
- Python docs — argparse module

### Practice Exercises
1. Build a CRUD CLI with typer and subcommands
2. Add rich tables, progress bars, and panels to an existing CLI
3. Implement --json flag that outputs machine-readable data
4. Write CLI tests with CliRunner
5. Package a CLI as an installable tool with entry points
