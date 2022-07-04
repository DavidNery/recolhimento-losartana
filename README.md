# Recolhimento losartana
O site <strong>Recolhimento Losartana</strong> foi desenvolvido por um
        <a href="https://www.instagram.com/davidmnery/" target="_blank">estudante de medicina</a>
        com o intuito de
        facilitar a vida dos profissionais de saúde e da população em geral ao pesquisar sobre os
        <a href="https://www.gov.br/anvisa/pt-br/assuntos/noticias-anvisa/2022/anvisa-determina-recolhimento-de-lotes-do-anti-hipertensivo-losartana/losartana.recolhimento"
          target="_blank">lotes anunciados</a>
        pela anvisa para recolhimento.

## Preview
![image](./.github/index.png)

## Como eu extraí os lotes?
Para extrair os lotes do PDF, utilizei a linguagem Java + a biblioteca [PDFBox](https://pdfbox.apache.org/), da Apache.

O código pode ser verificado abaixo
```java
PDFParser parser = new PDFParser(new RandomAccessBufferedFileInputStream("C:\\Users\\david\\Desktop\\losartana.pdf"));
parser.parse();
PDDocument doc = parser.getPDDocument();

HashMap<String, List<String>> lotes = new HashMap<>();

for (PDPage page : doc.getPages()) {
    PDRectangle rectangle = page.getBBox();

    PDFTextStripperByArea stripper = new PDFTextStripperByArea();
    stripper.addRegion("page", new Rectangle2D.Double(0, 0, rectangle.getWidth(), rectangle.getHeight()));
    stripper.extractRegions(page);
    String[] text = stripper.getTextForRegion("page").split("\n");

    // BRAINFARMA INDÚSTRIA QUÍMICA E FARMACÊUTICA S.A B20J2231 Recolhimento
    for (String s : text) {
        if (s.startsWith("Recolhimentos") || s.startsWith("Razão Social")) continue;

        String fabricanteLote = s.substring(0, s.lastIndexOf(' '));
        String fabricante, lote;

        if (fabricanteLote.startsWith("BRAINFARMA")) {
            fabricante = "BRAINFARMA INDÚSTRIA QUÍMICA E FARMACÊUTICA S.A";
            lote = fabricanteLote.substring(47);
        } else {
            int lastIndexSpace = fabricanteLote.lastIndexOf(' ');
            fabricante = fabricanteLote.substring(0, lastIndexSpace);
            lote = fabricanteLote.substring(lastIndexSpace + 1);
        }

        lotes.compute(fabricante, (key, value) -> {
            if (value == null) value = new ArrayList<>();

            value.add(lote);
            return value;
        });
    }
}

doc.close();

JsonObject fabricantes = new JsonObject();
lotes.forEach((key, value) -> {
    JsonArray fabricanteLotes = new JsonArray(value.size());
    value.forEach(fabricanteLotes::add);
    fabricantes.add(key, fabricanteLotes);
});

File f = new File("C:\\Users\\david\\Desktop\\losartana.json");
if (f.exists()) f.delete();
f.createNewFile();

FileWriter fileWriter = new FileWriter(f);
BufferedWriter writer = new BufferedWriter(fileWriter);
writer.write(fabricantes.toString());
writer.close();
```