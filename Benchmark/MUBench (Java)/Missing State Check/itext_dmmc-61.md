## Repository

URL: https://github.com/itext/itextpdf.git

Commit: adda8eb38978eeea6e86aca64ae746754c103dae

## API

java.util.Iterator

## Caller

```java
package com.itextpdf.awt;

public class PdfGraphics2D extends Graphics2D {

    private static final int FILL = 1;
    private static final int STROKE = 2;
    private static final int CLIP = 3;
    private BasicStroke strokeOne = new BasicStroke(1);

    private static final AffineTransform IDENTITY = new AffineTransform();

    private Font font;
    private BaseFont baseFont;
    private float fontSize;
    private AffineTransform transform;
    private Paint paint;
    private Color background;
    private float width;
    private float height;

    private Area clip;

    private RenderingHints rhints = new RenderingHints(null);

    private Stroke stroke;
    private Stroke originalStroke;

    private PdfContentByte cb;

    /** Storage for BaseFont objects created. */
    private HashMap<String, BaseFont> baseFonts;

    private boolean disposeCalled = false;

    private FontMapper fontMapper;

    private boolean drawImage(Image img, Image mask, AffineTransform xform, Color bgColor, ImageObserver obs) {
        if (xform==null)
            xform = new AffineTransform();
        else
            xform = new AffineTransform(xform);
        xform.translate(0, img.getHeight(obs));
        xform.scale(img.getWidth(obs), img.getHeight(obs));

        AffineTransform inverse = this.normalizeMatrix();
        AffineTransform flipper = AffineTransform.getScaleInstance(1,-1);
        inverse.concatenate(xform);
        inverse.concatenate(flipper);

        double[] mx = new double[6];
        inverse.getMatrix(mx);
        if (currentFillGState != 255) {
            PdfGState gs = fillGState[255];
            if (gs == null) {
                gs = new PdfGState();
                gs.setFillOpacity(1);
                fillGState[255] = gs;
            }
            cb.setGState(gs);
        }

        try {
            com.itextpdf.text.Image image = null;
            if(!convertImagesToJPEG){
                image = com.itextpdf.text.Image.getInstance(img, bgColor);
            }
            else{
                BufferedImage scaled = new BufferedImage(img.getWidth(null), img.getHeight(null), BufferedImage.TYPE_INT_RGB);
                Graphics2D g3 = scaled.createGraphics();
                g3.drawImage(img, 0, 0, img.getWidth(null), img.getHeight(null), null);
                g3.dispose();

                ByteArrayOutputStream baos = new ByteArrayOutputStream();
                ImageWriteParam iwparam = new JPEGImageWriteParam(Locale.getDefault());
                iwparam.setCompressionMode(ImageWriteParam.MODE_EXPLICIT);
                iwparam.setCompressionQuality(jpegQuality);//Set here your compression rate
                ImageWriter iw = ImageIO.getImageWritersByFormatName("jpg").next();
                ImageOutputStream ios = ImageIO.createImageOutputStream(baos);
                iw.setOutput(ios);
                iw.write(null, new IIOImage(scaled, null, null), iwparam);
                iw.dispose();
                ios.close();

                scaled.flush();
                scaled = null;
                image = com.itextpdf.text.Image.getInstance(baos.toByteArray());

            }
            if (mask!=null) {
                com.itextpdf.text.Image msk = com.itextpdf.text.Image.getInstance(mask, null, true);
                msk.makeMask();
                msk.setInverted(true);
                image.setImageMask(msk);
            }
            cb.addImage(image, (float)mx[0], (float)mx[1], (float)mx[2], (float)mx[3], (float)mx[4], (float)mx[5]);
            Object url = getRenderingHint(HyperLinkKey.KEY_INSTANCE);
            if (url != null && !url.equals(HyperLinkKey.VALUE_HYPERLINKKEY_OFF)) {
            	PdfAction action = new  PdfAction(url.toString());
                cb.setAction(action, (float)mx[4], (float)mx[5], (float)(mx[0]+mx[4]), (float)(mx[3]+mx[5]));
            }
        } catch (Exception ex) {
            throw new IllegalArgumentException(ex);
        }
        if (currentFillGState != 255) {
            PdfGState gs = fillGState[currentFillGState];
            cb.setGState(gs);
        }
        return true;
    }
}

```

## Description

On line 1599, `Iterator.next()` is invoked without prior check that there are sufficiently many elements. (Same finding as `mudetect-1` and `mudetectxp-1`)


## Rule

Missing State Check

## Response

[]
Explanation: The provided source code snippet's method "drawImage" does not use the API classes indicated in the input ("java.util.Iterator"). There is no misuse of APIs since the concerned API is not used at all. Hence, the return is an empty list as there are no issues found.

## Review

Found the expected misuse? **False**

