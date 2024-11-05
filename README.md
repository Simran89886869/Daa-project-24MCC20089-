# Daa-project-24MCC20089-

import tkinter as tk
from tkinter import messagebox, ttk
import networkx as nx
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg

# Union-Find class for cycle detection
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n

    def find(self, u):
        if self.parent[u] != u:
            self.parent[u] = self.find(self.parent[u])
        return self.parent[u]

    def union(self, u, v):
        root_u = self.find(u)
        root_v = self.find(v)
        if root_u != root_v:
            if self.rank[root_u] > self.rank[root_v]:
                self.parent[root_v] = root_u
            elif self.rank[root_u] < self.rank[root_v]:
                self.parent[root_u] = root_v
            else:
                self.parent[root_v] = root_u
                self.rank[root_u] += 1

# Function to apply Kruskal's Algorithm
def kruskal_mst(graph):
    edges = sorted(graph.edges(data=True), key=lambda x: x[2]['weight'])
    uf = UnionFind(len(graph.nodes))

    mst = []
    city_to_index = {city: i for i, city in enumerate(graph.nodes)}

    for u, v, data in edges:
        if uf.find(city_to_index[u]) != uf.find(city_to_index[v]):
            uf.union(city_to_index[u], city_to_index[v])
            mst.append((u, v, data['weight']))
            if len(mst) == len(graph.nodes) - 1:
                break

    return mst

# Main GUI application class
class KruskalApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Road Construction Cost Optimizer")
        self.center_window(1000, 600)  # Center window with specified width and height
        self.root.configure(bg="#F0F2F5")

        # Main Frame
        main_frame = tk.Frame(root, bg="#F0F2F5", padx=20, pady=20)
        main_frame.pack(expand=True, fill='both')

        # Title Label
        title_label = tk.Label(main_frame, text="Road Construction Cost Optimizer",
                               font=("Helvetica", 18, "bold", "italic"), bg="#394867", fg="white")
        title_label.grid(row=0, column=0, columnspan=2, pady=(0, 20), sticky="ew")

        # Input Frame
        input_frame = tk.Frame(main_frame, bg="#F0F2F5")
        input_frame.grid(row=1, column=0, sticky="nsew", padx=(0, 20))

        tk.Label(input_frame, text="City 1:", bg="#F0F2F5", font=("Arial", 12)).grid(row=0, column=0, sticky="e", padx=5)
        self.city1_entry = tk.Entry(input_frame, width=20, font=("Arial", 12), bg="#ffffff")
        self.city1_entry.grid(row=0, column=1, padx=10)

        tk.Label(input_frame, text="City 2:", bg="#F0F2F5", font=("Arial", 12)).grid(row=1, column=0, sticky="e", padx=5)
        self.city2_entry = tk.Entry(input_frame, width=20, font=("Arial", 12), bg="#ffffff")
        self.city2_entry.grid(row=1, column=1, padx=10)

        tk.Label(input_frame, text="Construction Cost:", bg="#F0F2F5", font=("Arial", 12)).grid(row=2, column=0, sticky="e", padx=5)
        self.cost_entry = tk.Entry(input_frame, width=20, font=("Arial", 12), bg="#ffffff")
        self.cost_entry.grid(row=2, column=1, padx=10)

        # Button Frame
        button_frame = tk.Frame(main_frame, bg="#F0F2F5")
        button_frame.grid(row=2, column=0, sticky="nsew", pady=15)

        add_edge_button = tk.Button(button_frame, text="Add Road", command=self.add_road, bg="#5cb85c", fg="white",
                                    font=("Arial", 12, "bold"), width=12)
        add_edge_button.grid(row=0, column=0, padx=5)

        find_mst_button = tk.Button(button_frame, text="Optimize Road Network", command=self.find_mst,
                                    bg="#0275d8", fg="white", font=("Arial", 12, "bold"), width=18)
        find_mst_button.grid(row=0, column=1, padx=5)

        reset_button = tk.Button(button_frame, text="Reset", command=self.reset, bg="#d9534f", fg="white",
                                 font=("Arial", 12, "bold"), width=12)
        reset_button.grid(row=0, column=2, padx=5)

        # Treeview to display the list of roads
        self.road_list = ttk.Treeview(main_frame, columns=("City1", "City2", "Cost"), show="headings", height=5)
        self.road_list.grid(row=1, column=1, sticky="nsew")
        self.road_list.heading("City1", text="City 1")
        self.road_list.heading("City2", text="City 2")
        self.road_list.heading("Cost", text="Construction Cost")

        # Canvas for matplotlib plot
        self.figure = plt.Figure(figsize=(5, 5), dpi=100)
        self.canvas = FigureCanvasTkAgg(self.figure, main_frame)
        self.canvas.get_tk_widget().grid(row=3, column=1, sticky="nsew")

        # Initialize graph
        self.graph = nx.Graph()

    def center_window(self, width, height):
        """ Center the main window on the screen """
        screen_width = self.root.winfo_screenwidth()
        screen_height = self.root.winfo_screenheight()
        x = (screen_width // 2) - (width // 2)
        y = (screen_height // 2) - (height // 2)
        self.root.geometry(f"{width}x{height}+{x}+{y}")

    def add_road(self):
        city1 = self.city1_entry.get().strip()
        city2 = self.city2_entry.get().strip()
        try:
            cost = float(self.cost_entry.get())
            if city1 and city2 and city1 != city2:
                self.graph.add_edge(city1, city2, weight=cost)
                self.road_list.insert("", "end", values=(city1, city2, f"${cost:.2f}"))
                messagebox.showinfo("Success", f"Road between {city1} and {city2} with cost ${cost:.2f} added!")
            else:
                messagebox.showerror("Input Error", "Please enter two different city names.")
        except ValueError:
            messagebox.showerror("Input Error", "Please enter a valid number for construction cost.")
        finally:
            self.city1_entry.delete(0, tk.END)
            self.city2_entry.delete(0, tk.END)
            self.cost_entry.delete(0, tk.END)

    def find_mst(self):
        if not self.graph.edges:
            messagebox.showerror("Error", "No roads have been added.")
            return

        mst_edges = kruskal_mst(self.graph)
        self.figure.clear()
        ax = self.figure.add_subplot(111)
        pos = nx.spring_layout(self.graph)

        nx.draw(self.graph, pos, ax=ax, with_labels=True, node_color='skyblue', node_size=700, font_size=10)
        nx.draw_networkx_edge_labels(self.graph, pos, ax=ax,
                                     edge_labels={(u, v): f"${data['weight']}" for u, v, data in self.graph.edges(data=True)})

        mst_graph = nx.Graph()
        mst_graph.add_weighted_edges_from(mst_edges)
        nx.draw(mst_graph, pos, ax=ax, edge_color='green', width=2)

        ax.set_title("Road Construction Cost Optimizer", fontsize=12, fontweight="bold")
        self.canvas.draw()

    def reset(self):
        self.graph.clear()
        self.figure.clear()
        self.canvas.draw()
        self.city1_entry.delete(0, tk.END)
        self.city2_entry.delete(0, tk.END)
        self.cost_entry.delete(0, tk.END)
        self.road_list.delete(*self.road_list.get_children())
        messagebox.showinfo("Reset", "All data has been cleared.")

# Run the application
if __name__ == "__main__":
    root = tk.Tk()
    app = KruskalApp(root)
    root.mainloop()
