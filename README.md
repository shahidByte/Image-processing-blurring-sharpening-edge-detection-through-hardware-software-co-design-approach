import cv2
2 import numpy as np
3
4 HEX_FILE = " image_in .mem"
5 IMG_W , IMG_H = 100 , 100
6
7 def convert_custom_image () :
8 img = cv2 . imread (" landscape . jpeg ", cv2 . IMREAD_GRAYSCALE )
9 img_resized = cv2 . resize ( img , ( IMG_W , IMG_H ) )
10
11 with open ( HEX_FILE , ’w’) as f :
12 for row in range ( IMG_H ) :
13 for col in range ( IMG_W ) :
1
14 pixel = img_resized [ row , col ]
15 f . write ( f"{ pixel :02x}\n")
16 print (" Created image_in .mem")
//Python Pre-Processing Scrip//

//Hardware Accelerator//

1 module image_filter (
2 input [1:0] mode , // 0= Blur , 1= Sharpen , 2= Edge
3 input [7:0] p1 , p2 , p3 , // 3x3 Window Inputs
4 input [7:0] p4 , p5 , p6 ,
5 input [7:0] p7 , p8 , p9 ,
6 output reg [7:0] out_pixel
7 ) ;
8 parameter THRESHOLD = 30;
9 integer sum , sharp_val , gx , gy , mag ;
10
11 always @ (*) begin
12 if ( mode == 0) begin // Blur
13 sum = p1 + p2 + p3 + p4 + p5 + p6 + p7 + p8 + p9 ;
14 out_pixel = sum / 9;
15 end
16 else if ( mode == 1) begin // Sharpen
17 sharp_val = ( p5 * 5) - ( p2 + p4 + p6 + p8 ) ;
18 if ( sharp_val < 0) out_pixel = 0;
19 else if ( sharp_val > 255) out_pixel = 255;
20 else out_pixel = sharp_val [7:0];
21 end
22 else begin // Edge
23 gx = ( p3 + 2* p6 + p9 ) - ( p1 + 2* p4 + p7 ) ;
24 gy = ( p7 + 2* p8 + p9 ) - ( p1 + 2* p2 + p3 ) ;
25 mag = ( gx < 0 ? - gx : gx ) + ( gy < 0 ? - gy : gy ) ;
26
27 if ( mag < THRESHOLD ) out_pixel = 0;
28 else if ( mag > 255) out_pixel = 255;
29 else out_pixel = mag [7:0];
30 end
31 end
32 endmodule
//Python Visualization Script//
1 import cv2
2 import numpy as np
3 import matplotlib . pyplot as plt
4 import os
5
6 OUTPUT_FILE = " combined_output .mem"
7 INPUT_FILE = " landscape . jpeg "
8 IMG_W = 100
9 IMG_H = 100
10
11 def view_combined_results () :
12 if not os . path . exists ( OUTPUT_FILE ) :
13 return
14
15 with open ( OUTPUT_FILE , ’r’) as f :
16 lines = f . readlines ()
17
18 pixels = [int( line . strip () , 16) for line in lines if line . strip () ]
19 target_h = IMG_H - 2
20 target_w = IMG_W - 2
21 one_img_size = target_h * target_w
22
23 titles = [" Blur ", " Sharpen ", " Edge Detection "]
24
25 plt . figure ( figsize =(20 , 5) )
26
27 plt . subplot (1 , 4 , 1)
28 if os . path . exists ( INPUT_FILE ) :
29 img_in = cv2 . imread ( INPUT_FILE , cv2 . IMREAD_GRAYSCALE )
30 if img_in is not None :
31 img_in = cv2 . resize ( img_in , ( IMG_W , IMG_H ) )
32 plt . imshow ( img_in , cmap =’gray ’)
33 plt . title (" Original Input ")
34 plt . axis (’off ’)
35
36 for i in range (3) :
37 start_idx = i * one_img_size
38 end_idx = start_idx + one_img_size
39 slice_pixels = pixels [ start_idx : end_idx ]
40
41 if len( slice_pixels ) < one_img_size :
42 slice_pixels += [0] * ( one_img_size - len( slice_pixels ) )
43
44 try:
45 img_out = np . array ( slice_pixels , dtype = np . uint8 ) . reshape ((
target_h , target_w ) )
46 fname = f" output_ { titles [i]. lower (). replace ( ’ ’, ’_ ’)}. jpg"
47 cv2 . imwrite ( fname , img_out )
48
49 plt . subplot (1 , 4 , i +2)
4
50 plt . imshow ( img_out , cmap =’gray ’)
51 plt . title ( f" Hardware : { titles [i]}")
52 plt . axis (’off ’)
53 except Exception as e :
54 print ( f" Error processing image {i}: {e}")
55
56 plt . tight_layout ()
57 plt . show ()
58
59 if __name__ == " __main__ ":
60 view_combined_results ()

