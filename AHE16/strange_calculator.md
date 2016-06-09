# Strange Calculator (100)
## Tools
- jadx-gui

This one has been our first challenge. By opening the apk with `jadx-gui` and having our first look at the file `/res/layout/activity_main.xml`, we can see that the method `createBackground(View v)` is getting called by pressing the button.

```
<Button android:id="@+id/btnCalc" android:layout_width="wrap_content" android:layout_height="wrap_content" android:layout_marginTop="53dp" android:text="Calculate" android:layout_below="@+id/txtExpression" android:layout_alignLeft="@+id/txtExpression" android:onClick="createBackground" />
```

The calculation we entered is evaluated by a class named `Parser`.
```java
public void createBackground(View v) {
  String s = ((EditText) this.view).getText().toString();
  try {
    TextView result = (TextView) findViewById(R.id.lblResult);
    result.setText("");
    result.setText(String.valueOf(Parser.eval(s)));
  } catch (Exception e) {
    Toast.makeText(this, e.getMessage(), 1).show();
  }
}
```

In there we found dead code in `parseExpression`, and it seemed to be very suspicious.
```java
double parseExpression() {
  [...]
  if (v > 100.0d) {
    throw new RuntimeException("The number is too large. Please buy the full version!");
  }
  if (v > 100.0d) {
    int[] flarry = new int[]{1400, 1393, 1404, 1288, 1295, 1346, 1395, 1368, 1359, 1368, 1382, 1293, 1367, 1368, 1365, 1344, 1354, 1288, 1354, 1382, 1288, 1354, 1382, 1355, 1293, 1357, 1361, 1290, 1355, 1382, 1290, 1368, 1354, 1344, 1382, 1288, 1354, 1367, 1357, 1382, 1288, 1357, 1348};
    for (int i : flarry) {
      Log.d("SUPER OUTPUT", Integer.toString(i ^ 1337));
    }
  }
  [...]
}
```

By extracting the code and running it, we got the flag `AHE16{Java_4nalys1s_1s_r4th3r_3asy_1snt_1t}`
