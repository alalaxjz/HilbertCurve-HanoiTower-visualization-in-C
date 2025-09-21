# HilbertCurve-HanoiTower-visualization-in-C
Recursive visualization of Hilbert Curve and HanoiTower using two distinct algorithmic approaches.<br>
Includes C++ implementations, visual results, and design commentary for each method.<br><br>
Question: To visualize HIlbertCurve, if input = 1:
<pre><br>+  +
|  |
+--+<br></pre>
If input = 2:
<pre><br>+--+  +--+
   |  |   
+--+  +--+
|        |
+  +--+  +
|  |  |  |
+--+  +--+<br></pre>
If input = 3:
<pre><br>+  +--+--+  +--+--+  +
|  |     |  |     |  |
+--+  +--+  +--+  +--+
      |        |      
+--+  +--+  +--+  +--+
|  |     |  |     |  |
+  +--+--+  +--+--+  +
|                    |
+--+  +--+--+--+  +--+
   |  |        |  |   
+--+  +--+  +--+  +--+
|        |  |        |
+  +--+  +  +  +--+  +
|  |  |  |  |  |  |  |
+--+  +--+  +--+  +--+<br></pre>

AS we can see each level contains four small level, the top left one is rotated counter-clockwise 90 degrees. The top right is rotated clockwise 90 degrees. The bottom left and right ones doesn't rotate.<br>

The first approach is the simplest. First we make a canvas with calculated width and length:

<pre><br>vector<string> canvas(height, string(width, ' '));<br></pre>

Note that we set up left corner to be x=0, y=0. Then we create a function(auto &canvas, int level,int x, int y) as well as function_counter and function_clockwise to represent the rotated situation.<br>

Inside the function we: 

<pre><br>{
    function_counter(canvas, level-1, x,y);
    x+=(1<<level-1);
    function_clockwise(canvas, level-1, x,y);
    ...
    //now we add the connecting line
}
<br></pre>

So we call one time with the correct direction, move to the down left of the next small square, call, move, call, move. After calling four times we then move x and y to the connecting point to add "-" or "|". Note that in the function we should add:

<pre><br>if (level==1){
    canvas[y][x] = "+";
    canvas[y+1][x] = "|";
    canvas[y+2][x] = "+";
    ...
    return;
    }<br></pre>

so when dealing with level 3 we actually calls level 2 for 4 times and each will call level 1 for four times. Because level 3 doesn't rotate, we can the normal function(3,0,0). Remember that we create three functions so that each function the calling sequence schould be different and the shape inside the canvas in the if statement(level==1) should be different. normal, counter, clockwise, respectively.<br>

After all the loop, we can now cout the canvas.

<pre><br>// trim trailing spaces and output
    for (int r = 0; r < height; ++r) {
        int last = width - 1;
        while (last >= 0 && canvas[r][last] == ' ') --last;
        if (last < 0) cout << '\n';
        else cout << canvas[r].substr(0, last+1) << '\n';
    }

    return 0;<br></pre>

I didn't use this method but rather discuss with copilot to approach with another way:

<pre><br>#include <bits/stdc++.h>
using namespace std;

// d2xy for Hilbert curve (standard iterative bitwise algorithm).
// m = 2^order (side length).
void d2xy(int m, long long d, int &x, int &y) {
    x = 0; y = 0;
    long long t = d;
    for (int s = 1; s < m; s <<= 1) {
        int rx = (t / 2) & 1;
        int ry = (t ^ rx) & 1;
        if (ry == 0) {
            if (rx == 1) {
                x = s - 1 - x;
                y = s - 1 - y;
            }
            std::swap(x, y);
        }
        x += s * rx;
        y += s * ry;
        t /= 4;
    }
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int level;
    cin >> level;

    int m = 1 << level;
    long long total = 1LL * m * m;

    // spacing rules:
    // horizontal adjacent points must leave two '-' chars => dx = 3
    // vertical adjacent points must leave one '|' char => dy = 2
    const int dx = 3;
    const int dy = 2;
    int width = (m - 1) * dx + 1;
    int height = (m - 1) * dy + 1;

    // allocate canvas
    vector<string> canvas(height, string(width, ' '));
    // generate Hilbert sequence and draw on the fly to reduce memory
    // but we need previous point to draw edge: store prev point
    long long d = 0;
    int px = 0, py = 0;
    bool first = true;

    auto putPlus = [&](int gx, int gy) {
        int cx = gx * dx;
        int cy = gy * dy;
        canvas[cy][cx] = '+';
    };

    for (d = 0; d < total; ++d) {
        int x, y;
        d2xy(m, d, x, y);
        if (first) {
            putPlus(x, y);
            px = x; py = y;
            first = false;
            continue;
        }
        // place plus at current
        putPlus(x, y);
        // draw edge between (px,py) and (x,y)
        int cx1 = px * dx, cy1 = py * dy;
        int cx2 = x  * dx, cy2 = y  * dy;
        if (cy1 == cy2 && abs(cx1 - cx2) == dx) {
            int l = min(cx1, cx2), r = max(cx1, cx2);
            for (int xx = l+1; xx < r; ++xx) canvas[cy1][xx] = '-';
            canvas[cy1][l] = '+'; canvas[cy1][r] = '+';
        } else if (cx1 == cx2 && abs(cy1 - cy2) == dy) {
            int t = min(cy1, cy2), b = max(cy1, cy2);
            for (int yy = t+1; yy < b; ++yy) canvas[yy][cx1] = '|';
            canvas[t][cx1] = '+'; canvas[b][cx1] = '+';
        } 
        px = x; py = y;
    }

    // trim trailing spaces and output
    for (int r = 0; r < height; ++r) {
        int last = width - 1;
        while (last >= 0 && canvas[r][last] == ' ') --last;
        if (last < 0) cout << '\n';
        else cout << canvas[r].substr(0, last+1) << '\n';
    }

    return 0;
}<br></pre>

Notice that we count all the point that need to be discussed:total = 1LL * m * m. for (d = 0; d < total; ++d). Since every square contains four small square we define every two digits of d in binary(00,01,10,11)to be the calculated and consider whether to flip and swap it. In this situation:

<pre><br>
00|01
--+--
11|10
<br></pre>

And we calculate rx(flip), ry(swap) in d2xy() function. rx and ry also determines whether we should move it right and down an unit. The d2xy funxtion determines the x and y for each d from 0 to total-1. And in the main function we see how x, y differs from previous (d-1) px(previous x) py (previous y) to choose how to connect them. Notice that when d is put into d2xy we traverse each smaller level and get the two digits with corresponding unit s to decide how far we should move it. Traverse into next level, s*=2 and first consider where it is in this level, whether we should flip and swap it or not, then move them by the unit s. After traversing each level we then know where d should be and connect all of them to the previous one. Finally we cout it.<br>

For HanoiTower:IF input =1:

<pre><br>  @  
 / \ 
@   @<br></pre>

2:
<pre><br>      @      
     / \     
    @   @    
   /     \   
  @       @  
   \     /   
@---@   @---@<br></pre>

3:

<pre><br>              @
             / \
            @   @
           /     \
          @       @
           \     /
        @---@   @---@
       /             \
      @               @
     /                 \
    @---@           @---@
         \         /
  @       @       @       @
 / \       \     /       / \
@   @---@---@   @---@---@   @ <br></pre>

Its similar and because the rotate can be difficult I use the first method:

<pre><br>#include <bits/stdc++.h>
using namespace std;

void draw_level_for_up(int level, int x, int y, vector<string> &canvas);
void draw_level_for_left(int level, int x, int y, vector<string> &canvas);
void draw_level_for_right(int level, int x, int y, vector<string> &canvas);


void draw_level_for_up(int level, int x, int y,vector<string> &canvas){
    if(level ==0){
        canvas[y][x] = '@';
        return;
    }
    if(level == 1){
        canvas[y][x] = '@';
        canvas[y-1][x+1] = '/';
        canvas[y-2][x+2] = '@';
        canvas[y-1][x+3] = '\\';
        canvas[y][x+4] = '@';
        return;
    }
    draw_level_for_left(level-1,x,y,canvas);
    x += (1 << level) - 1;
    y -= (1 << level) - 1;
    canvas[y][x] = '/';
    x+=1, y-=1;
    draw_level_for_up(level-1,x,y,canvas);
    x += (1 << level);
    y += (1 << level);
    draw_level_for_right(level-1,x,y,canvas);
    x += (1 << level) - 3;
    y -= (1 << level) - 1;
    canvas[y][x] = '\\';
}

void draw_level_for_right(int level, int x, int y,vector<string> &canvas){
    if(level ==0){
        canvas[y][x] = '@';
        return;
    }
    if(level == 1){
        canvas[y][x] = '@';
        canvas[y-1][x+1] = '/';
        canvas[y-2][x+2] = '@';
        canvas[y][x+1] = '-';
        canvas[y][x+2] = '-';
        canvas[y][x+3] = '-';
        canvas[y][x+4] = '@';
        return;
    }
    draw_level_for_right(level-1,x,y,canvas);
    x += (1 << level) - 1;
    y -= (1 << level) - 1;
    canvas[y][x] = '/';
    x+=1, y-=1;
    draw_level_for_left(level-1,x,y,canvas);
    x += (1 << level) - 3;
    y += (1 << level);
    canvas[y][x] = '-';
    x+=1;
    canvas[y][x] = '-';
    x+=1;
    canvas[y][x] = '-';
    x+=1;
    draw_level_for_up(level-1,x,y,canvas);
}

void draw_level_for_left(int level, int x, int y,vector<string> &canvas){
    if(level ==0){
        canvas[y][x] = '@';
        return;
    }
    if(level == 1){
        canvas[y][x] = '@';
        canvas[y-1][x+3] = '\\';
        canvas[y-2][x+2] = '@';
        canvas[y][x+1] = '-';
        canvas[y][x+2] = '-';
        canvas[y][x+3] = '-';
        canvas[y][x+4] = '@';
        return;
    }
    draw_level_for_up(level-1,x,y,canvas);
    x += (1 << level)*2 - 3;
    canvas[y][x] = '-';
    x+=1;
    canvas[y][x] = '-';
    x+=1;
    canvas[y][x] = '-';
    x+=1;
    draw_level_for_left(level-1,x,y,canvas);
    x -= (1 << level);
    y -= (1 << level);
    draw_level_for_right(level-1,x,y,canvas);
    x += (1 << level)*2-3;
    y += 1;
    canvas[y][x] = '\\';

    
}

int main(){
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int level;
    cin >> level;

    int W = 4 * (1<<level) - 3;
    int H = 2 * (1<<level) - 1;
    int x =0 , y = H-1;

    vector<string> canvas;

    canvas.assign(H, string(W, ' '));

    draw_level_for_up(level, x,y,canvas);


    for(string &row : canvas){
        int last = (int)row.size() - 1;
        while(last >= 0 && row[last] == ' ') --last;
        row.resize(last+1);
        cout << row << "\n";
    }
    return 0;
}<br></pre>

The second method can also work but we should turn d into three digits system instead of two and each level consider one digits. The rotate can be difficult. I come up with starting from the down left corner(x=0,y = height-1). now d2xy instead of using x and y we us row and column to show the triangle(row 1 has 1 column, row 2 has 2 column). For rotating we use the formula, something like r = size-r', y = size - r' - y'. The formula can be searched online. After traversing through each level and its unit we then change it back to the xy system on the canva and cout.

