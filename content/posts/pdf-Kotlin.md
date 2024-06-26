+++
title = 'Creating Shareable PDF Reports with Kotlin and iText from RecyclerView Data'
date = 2024-05-28T13:00:23-03:00
draft = false
+++

## Motivation

Tables are a cornerstone of data presentation in mobile apps. Android's RecyclerView makes it easy to create dynamic tables with large datasets. However, exporting this data to a PDF or other file format for sharing can be challenging.
This post will address this challenge by showing you how to leverage Kotlin and iText to automate the creation and sharing of reports from your RecyclerView data, ensuring proper formatting for A4 pages.

## Packages

For this feature, we will be using iText v5.5.10 and Kotlin v1.8.0.

You can add iText dependency on your project's `build.gradle` file:

```kotlin
dependencies {
    implementation 'com.itextpdf:itextg:5.5.10'
}
```

## Usage

Firstly, we are going to create a class for managing the PDF file and a generic creation of headers and cells for the table using iText tools:

```kotlin
open class PdfGenerator {
  /**
   * Creates a PdfPCell for the table header with specified content, height,
   * and optional border widths.
   */
  fun createHeaderCell(
    content: String?,
    height: Float,
    borderWidthLeft: Float = 0f,
    borderWidthRight: Float = 0f
  ): PdfPCell {
    val cell =
      PdfPCell(Phrase(content, FontFactory.getFont("arial", 12f, Font.BOLD, BaseColor.BLACK)))
    cell.fixedHeight = height
    cell.borderWidthTop = 0.5f
    cell.borderWidthBottom = 0.5f
    cell.borderWidthLeft = borderWidthLeft
    cell.borderWidthRight = borderWidthRight
    cell.horizontalAlignment = Element.ALIGN_CENTER
    return cell
  }

  /** 
   * Creates a PdfPCell for the table header with specified content, height, 
   * and optional border widths.
   */
  fun createCell(
    content: String?,
    alignment: Int,
    borderWidthLeft: Float = 0f,
    borderWidthRight: Float = 0f
  ): PdfPCell {
    val cell =
      PdfPCell(Phrase(content, FontFactory.getFont("arial", 10f, Font.NORMAL, BaseColor.BLACK)))
    cell.borderWidthTop = 0f
    cell.borderWidthBottom = 0.5f
    cell.borderWidthLeft = borderWidthLeft
    cell.borderWidthRight = borderWidthRight
    cell.horizontalAlignment = alignment
    return cell
  }
}
```

Here, we are using `PdfPCell()` for creating each cell of the header and the body for our table. The only lines on the body of the table will be the bottom ones, delimiting the different rows, so we can create a nice spacing table without too many lines. The header, on the other hand, will be enclosed on a rectangle.

Next, we will create the table using the data passed to the Recycler View on our app, so we can iterate over it and create each row of the table. For that matter, we are extending our previous class so we can use the functions already coded.

Firstly, we are going to calculate how many pages we need to create:

```kotlin
 val ITEMS_PER_PAGE = 40
  val remainder = data.items.size % ITEMS_PER_PAGE
  var pages = data.items.size / ITEMS_PER_PAGE

  if (remainder != 0) {
    pages += 1
  }

```

Having the number of pages, we can create our table:

```kotlin
class ReportPdf: PdfGenerator() {

  fun createReportPdf(
    context: Context,
    data: ReportResults,
    dateFrom: String,
    dateTo: String
  ) {
    val file = File(
      context.getExternalFilesDir(null),
      "exported.pdf"
    )
    val fileOutputStream = FileOutputStream(file)

    val document = Document()
    val writer = PdfWriter.getInstance(document, fileOutputStream)

    document.open()

    val sdf = SimpleDateFormat("dd/M/yyyy", Locale.getDefault())
    val currentDate = sdf.format(Date())

    val title = "Pdf Title"
    
    // Calculate number of pages based on data size
    val ITEMS_PER_PAGE = 40
    val remainder = data.items.size % ITEMS_PER_PAGE
    var pages = data.items.size / ITEMS_PER_PAGE

    if (remainder != 0) {
      pages += 1
    }
    
    // Loop through each page and create content
    for (i in 0 until pages) {
      val items =
        if (i == pages - 1 && remainder != 0) data.items.slice(i * ITEMS_PER_PAGE until data.items.size) else data.items.slice(
          i * ITEMS_PER_PAGE until (i + 1) * ITEMS_PER_PAGE
        )
      document.add(
        createTableHeader( // creates a header with a title and page number. Implementation not shown
          context, title, null, currentDate, i + 1
        )
      )
      document.add(
        createReportDataTable(
          context,
          items
        )
      )
      document.newPage()
    }

    document.close()
    savePdf(context, file) // Save or share the pdf. Implementation not shown
  }

  fun createReportDataTable(context: Context, items: List<ItemReport>): PdfPTable {
    // results table
    val table = PdfPTable(6)
    table.widthPercentage = 100f;

    // header
    table.addCell(
      createHeaderCell(
        context.getString(R.string.RP),
        30f,
        .5f,
        0f
      )
    )
    table.addCell(
      createHeaderCell(
        context.getString(R.string.nextDelivery),
        30f
      )
    )
    table.addCell(
      createHeaderCell(
        context.getString(R.string.nextDry),
        30f
      )
    )
    table.addCell(
      createHeaderCell(
        context.getString(R.string.condition),
        30f
      )
    )
    table.addCell(
      createHeaderCell(
        context.getString(R.string.lastLactancy),
        30f
      )
    )
    table.addCell(
      createHeaderCell(
        context.getString(R.string.bull),
        30f,
        0f,
        .5f
      )
    )
    // body
    for (item in items) {
      table.addCell(createCell(item.rp, Element.ALIGN_CENTER))
      table.addCell(createCell(item.nextDelivery, Element.ALIGN_CENTER))
      table.addCell(createCell(item.nextDry, Element.ALIGN_CENTER))
      table.addCell(createCell(item.condition, Element.ALIGN_CENTER))
      table.addCell(createCell(item.lastLactancy, Element.ALIGN_CENTER))
      table.addCell(createCell(item.bull, Element.ALIGN_CENTER))
    }

    return table
  }
}
```

And that’s all! We have made a PDF table on a A4 format based on our RecyclerView table.
