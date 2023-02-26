#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include <mpi.h>

#define WIDTH 1200
#define HEIGHT 1200
#define MAX_ITERATIONS 1000

int main(int argc, char** argv) {
    double x0, y0, x, y, x_temp;
    int i, j, k, n, rank, size;
    int color[WIDTH][HEIGHT];
    double scale_real = 3.0 / WIDTH;
    double scale_imag = 3.0 / HEIGHT;

    double start, end, comm_start, comm_end, comm_time;

    // Initialize MPI
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    // Determine the range of rows to be processed by this process
    int local_height = HEIGHT / size;
    int start_row = rank * local_height;
    int end_row = (rank + 1) * local_height;
    if (rank == size - 1) {
        end_row = HEIGHT;
    }

    // Allocate memory for the local color array
    int local_color[local_height][WIDTH];

    // Get the start timer
    start = MPI_Wtime();

    // Perform the Mandelbrot set calculation for each pixel in the range of rows
    for (i = start_row; i < end_row; i++) {
        for (j = 0; j < WIDTH; j++) {
            x0 = i * scale_real - 1.5;
            y0 = j * scale_imag - 1.5;
            x = 0.0;
            y = 0.0;
            k = 0;

            while (k < MAX_ITERATIONS && (x*x + y*y) < 4.0) {
                x_temp = x*x - y*y + x0;
                y = 2*x*y + y0;
                x = x_temp;
                k++;
            }

            if (k == MAX_ITERATIONS) {
                local_color[i-start_row][j] = 0;
            } else {
                local_color[i-start_row][j] = k;
            }
        }
    }
    // Record the start time of the communication
    comm_start = MPI_Wtime();

    // Gather the local color arrays into the global color array
    MPI_Gather(local_color, local_height * WIDTH, MPI_INT, color, local_height * WIDTH, MPI_INT, 0, MPI_COMM_WORLD);
    
    // Record the end time of the communication
    comm_end = MPI_Wtime();
    
    // Calculate the communication time
    comm_time = comm_end - comm_start;

    // Get the end time
    end = MPI_Wtime();

    // Output the result to a PPM file
    if (rank == 0) {
        FILE* fp = fopen("mandelbrot2.ppm", "wb");
        fprintf(fp, "P6\n%d %d\n255\n", WIDTH, HEIGHT);
        for (i = 0; i < WIDTH; i++) {
            for (j = 0; j < HEIGHT; j++) {
                n = color[i][j];
                fputc((n % 256), fp);
                fputc((n / 256 % 256), fp);
                fputc((n / 65536 % 256), fp);
            }
        }
        fclose(fp);

        // Output the time taken
        printf("Time taken: %f seconds\n", end - start);
        
        // output the communication time
        printf("Communication time taken: %f seconds\n", comm_time)
    }

    // Finalize MPI
    MPI_Finalize();

    return 0;
}
