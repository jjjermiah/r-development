## .Rprofile — Project-level startup script
## Author: Jermiah Joseph

# Set CRAN mirror unless already configured
if (getOption("repos")["CRAN"] == "@CRAN@") {
  options(repos = c(CRAN = "https://cran.rstudio.com/"))
}

# Only run the rest in interactive sessions
if (interactive()) {
  ## Check if a package is installed
  is_installed <- function(pkg) requireNamespace(pkg, quietly = TRUE)

  ## Install 'pak' if not available
  if (!is_installed("pak")) {
    message("📦 Installing 'pak'...")
    utils::install.packages(
      "pak",
      repos = sprintf(
        "https://r-lib.github.io/p/pak/devel/%s/%s/%s",
        .Platform$pkgType,
        R.Version()$os,
        R.Version()$arch
      )
    )
  }

  ## Load essential dev tools
  dev_tools <- c("devtools", "data.table", "usethis", "qs", "covr")

  # Install any missing dev tools in one go
  missing <- dev_tools[!vapply(dev_tools, is_installed, logical(1))]
  if (length(missing) > 0) {
    message("📦 Installing missing packages: ", paste(missing, collapse = ", "))
    pak::pkg_install(missing)
  }

  # Load all dev tools quietly
  suppressPackageStartupMessages(lapply(
    dev_tools,
    library,
    character.only = TRUE
  ))

  ## Increase command history size
  Sys.setenv(R_HISTSIZE = '100000')

  ## Create invisible environment for helpers
  .env <- new.env()

  # Quick aliases
  .env$s <- base::summary
  .env$h <- utils::head
  .env$asdt <- function(x, ...) data.table::as.data.table(x, ...)
  .env$ht <- function(d) rbind(head(d, 10), tail(d, 10))

  # Clipboard reader
  .env$read.cb <- function(...) {
    ismac <- Sys.info()[1] == "Darwin"
    if (!ismac) read.table(file = "clipboard", ...) else
      read.table(pipe("pbpaste"), ...)
  }

  # List objects and classes
  .env$lsa <- function() {
    obj_type <- function(x) class(get(x, envir = .GlobalEnv))
    df <- data.frame(sapply(ls(envir = .GlobalEnv), obj_type))
    df$object <- rownames(df)
    names(df)[1] <- "class"
    return(utils::unrowname(df))
  }

  # devtools shortcuts
  .env$di <- devtools::install
  .env$da <- devtools::load_all
  .env$db <- devtools::build
  .env$dd <- devtools::document
  .env$dc <- devtools::check

  .env$qi <- .env$qinstall <- function(quit = FALSE, clean = TRUE, ...) {
    if (requireNamespace("roxygen2", quietly = TRUE)) {
      roxygen2::roxygenise(clean = clean)
    }
    devtools::install(...)
    if (isTRUE(quit)) quit(save = "no")
  }

  # Quit without saving
  .env$qq <- .env$iquit <- function() q(save = "no")

  # pak shortcut
  .env$pki <- pak::pkg_install

  # List functions in a package
  .env$lsp <- function(package, all.names = FALSE, pattern) {
    package <- deparse(substitute(package))
    ls(
      pos = paste("package", package, sep = ":"),
      all.names = all.names,
      pattern = pattern
    )
  }

  # Set default usethis package description
  options(
    usethis.description = list(
      "Authors@R" = utils::person(
        "Jermiah",
        "Joseph",
        email = Sys.getenv("R_EMAIL", unset = "jermiahjoseph98@gmail.com"),
        role = c("aut", "cre")
      )
    ),
    usethis.destdir = "~/bhklab/jermiah/Bioconductor",
    usethis.overwrite = TRUE
  )

  # Auto-load dev package if DESCRIPTION is present
  if (file.exists("DESCRIPTION")) {
    devtools::load_all(quiet = TRUE)
  }

  # Notify about helper env
  message("ℹ️  .env loaded. Use .env$qi(), .env$cov(), .env$lsa(), etc.")

  # Attach helper env last
  attach(.env)
}
