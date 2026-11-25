import tkinter as tk
import customtkinter as ctk
import ast
import io
import sys
import tempfile
import os
import contextlib
import threading
import re
import math
import time


ctk.set_appearance_mode("dark")
ctk.set_default_color_theme("blue")


# ============================================================
# DATA STRUCTURES
# ============================================================

class Node:

    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None


class ListNode:

    def __init__(self, value):
        self.value = value
        self.next = None


class GraphNode:

    def __init__(self, value):
        self.value = value
        self.neighbors = []


# ============================================================
# MAIN APPLICATION
# ============================================================

class CodeFlow(ctk.CTk):

    def __init__(self):

        super().__init__()

        self.title(
            "CODEFLOW — AI DSA VISUALIZER"
        )

        self.geometry(
            "1550x900"
        )

        self.minsize(
            1200,
            720
        )

        self.configure(
            fg_color="#020617"
        )

        self.states = []
        self.current_step = 0
        self.running = False
        self.auto_running = False
        self.execution_finished = False

        # ADDED: AUTO PLAY SPEED
        self.play_speed = 450

        self.execution_type = "CODE"

        self.create_interface()

        self.after(
            300,
            self.draw_idle
        )

    # ========================================================
    # UI
    # ========================================================

    def create_interface(self):

        self.create_header()

        body = ctk.CTkFrame(
            self,
            fg_color="transparent"
        )

        body.pack(
            fill="both",
            expand=True,
            padx=14,
            pady=10
        )

        body.grid_columnconfigure(
            0,
            weight=3
        )

        body.grid_columnconfigure(
            1,
            weight=6
        )

        body.grid_columnconfigure(
            2,
            weight=3
        )

        body.grid_rowconfigure(
            0,
            weight=1
        )

        self.create_code_panel(
            body
        )

        self.create_visual_panel(
            body
        )

        self.create_execution_panel(
            body
        )

        self.create_bottom_bar()

    # ========================================================
    # HEADER
    # ========================================================

    def create_header(self):

        header = ctk.CTkFrame(
            self,
            height=65,
            fg_color="#050B16",
            corner_radius=0
        )

        header.pack(
            fill="x"
        )

        header.pack_propagate(
            False
        )

        ctk.CTkLabel(
            header,
            text="◈ CODEFLOW",
            font=(
                "Arial",
                25,
                "bold"
            ),
            text_color="#38BDF8"
        ).pack(
            side="left",
            padx=25
        )

        ctk.CTkLabel(
            header,
            text="INTERACTIVE CODE → VISUALIZATION ENGINE",
            font=(
                "Consolas",
                10,
                "bold"
            ),
            text_color="#64748B"
        ).pack(
            side="left"
        )

        # ADDED: PROFESSIONAL BRANDING
        ctk.CTkLabel(
            header,
            text="ANUPAM PAUL",
            font=(
                "Consolas",
                10,
                "bold"
            ),
            text_color="#A78BFA"
        ).pack(
            side="right",
            padx=(10, 20)
        )

        self.header_status = ctk.CTkLabel(
            header,
            text="● READY",
            font=(
                "Consolas",
                10,
                "bold"
            ),
            text_color="#34D399"
        )

        self.header_status.pack(
            side="right",
            padx=10
        )

    # ========================================================
    # CODE PANEL
    # ========================================================

    def create_code_panel(
        self,
        parent
    ):

        frame = ctk.CTkFrame(
            parent,
            fg_color="#080D18",
            corner_radius=15
        )

        frame.grid(
            row=0,
            column=0,
            sticky="nsew",
            padx=(0, 6)
        )

        ctk.CTkLabel(
            frame,
            text="💻 SOURCE CODE",
            font=(
                "Arial",
                13,
                "bold"
            ),
            text_color="#38BDF8"
        ).pack(
            anchor="w",
            padx=15,
            pady=(12, 5)
        )

        self.code = tk.Text(
            frame,
            bg="#030712",
            fg="#E2E8F0",
            insertbackground="#38BDF8",
            selectbackground="#164E63",
            font=(
                "Consolas",
                13
            ),
            bd=0,
            padx=14,
            pady=12,
            wrap="none"
        )

        self.code.pack(
            fill="both",
            expand=True,
            padx=8,
            pady=8
        )

        self.code.tag_configure(
            "current",
            background="#164E63",
            foreground="#FFFFFF"
        )

        self.code.tag_configure(
            "error",
            background="#7F1D1D",
            foreground="#FFFFFF"
        )

        sample = '''# STACK TEST

stack = []

stack.append(10)
stack.append(20)
stack.append(30)
stack.append(40)

print("Stack:", stack)

top = stack[-1]

popped = stack.pop()

stack.append(50)
stack.append(60)

print("Final Stack:", stack)
print("Top:", stack[-1])'''

        self.code.insert(
            "1.0",
            sample
        )

    # ========================================================
    # VISUAL PANEL
    # ========================================================

    def create_visual_panel(
        self,
        parent
    ):

        frame = ctk.CTkFrame(
            parent,
            fg_color="#080D18",
            corner_radius=15
        )

        frame.grid(
            row=0,
            column=1,
            sticky="nsew",
            padx=6
        )

        title = ctk.CTkFrame(
            frame,
            height=45,
            fg_color="transparent"
        )

        title.pack(
            fill="x",
            padx=12
        )

        title.pack_propagate(
            False
        )

        self.visual_title = ctk.CTkLabel(
            title,
            text="🎬 LIVE VISUALIZATION",
            font=(
                "Arial",
                13,
                "bold"
            ),
            text_color="#A78BFA"
        )

        self.visual_title.pack(
            side="left"
        )

        self.type_label = ctk.CTkLabel(
            title,
            text="CODE",
            font=(
                "Consolas",
                10,
                "bold"
            ),
            text_color="#64748B"
        )

        self.type_label.pack(
            side="right"
        )

        self.canvas = tk.Canvas(
            frame,
            bg="#030712",
            highlightthickness=0
        )

        self.canvas.pack(
            fill="both",
            expand=True,
            padx=8,
            pady=8
        )

        self.canvas.bind(
            "<Configure>",
            lambda event: self.refresh_visual()
        )

    # ========================================================
    # EXECUTION PANEL
    # ========================================================

    def create_execution_panel(
        self,
        parent
    ):

        frame = ctk.CTkFrame(
            parent,
            fg_color="#080D18",
            corner_radius=15
        )

        frame.grid(
            row=0,
            column=2,
            sticky="nsew",
            padx=(6, 0)
        )

        ctk.CTkLabel(
            frame,
            text="⚡ EXECUTION",
            font=(
                "Arial",
                13,
                "bold"
            ),
            text_color="#FBBF24"
        ).pack(
            anchor="w",
            padx=15,
            pady=12
        )

        self.concept = ctk.CTkLabel(
            frame,
            text="WAITING",
            font=(
                "Consolas",
                17,
                "bold"
            ),
            text_color="#64748B"
        )

        self.concept.pack(
            pady=(5, 12)
        )

        self.loop_canvas = tk.Canvas(
            frame,
            height=180,
            bg="#030712",
            highlightthickness=0
        )

        self.loop_canvas.pack(
            fill="x",
            padx=10
        )

        self.info = ctk.CTkTextbox(
            frame,
            fg_color="#030712",
            text_color="#94A3B8",
            font=(
                "Consolas",
                10
            )
        )

        self.info.pack(
            fill="both",
            expand=True,
            padx=10,
            pady=10
        )

    # ========================================================
    # BOTTOM BAR
    # ========================================================

    def create_bottom_bar(self):

        bar = ctk.CTkFrame(
            self,
            height=68,
            fg_color="#080D18",
            corner_radius=12
        )

        bar.pack(
            fill="x",
            padx=14,
            pady=(0, 12)
        )

        bar.pack_propagate(
            False
        )

        buttons = [
            ("▶ RUN", self.run),
            ("⏮", self.previous),
            ("STEP ▶", self.next),
            ("▶ AUTO PLAY", self.auto_play),
            ("↻ RESET", self.reset)
        ]

        for text, command in buttons:

            ctk.CTkButton(
                bar,
                text=text,
                command=command,
                width=110,
                height=38,
                corner_radius=9,
                font=(
                    "Arial",
                    11,
                    "bold"
                )
            ).pack(
                side="left",
                padx=5,
                pady=15
            )

        # ADDED: SPEED LABEL
        ctk.CTkLabel(
            bar,
            text="SPEED",
            font=(
                "Consolas",
                9,
                "bold"
            ),
            text_color="#64748B"
        ).pack(
            side="right",
            padx=(10, 5)
        )

        # ADDED: SPEED SLIDER
        self.speed_slider = ctk.CTkSlider(
            bar,
            from_=50,
            to=1000,
            width=150,
            command=self.change_speed
        )

        self.speed_slider.set(
            450
        )

        self.speed_slider.pack(
            side="right",
            padx=5
        )

        self.progress = ctk.CTkProgressBar(
            bar,
            width=240
        )

        self.progress.pack(
            side="right",
            padx=25
        )

        self.progress.set(
            0
        )

    # ========================================================
    # RUN
    # ========================================================

    def run(self):

        code = self.code.get(
            "1.0",
            "end-1c"
        )

        if not code.strip():
            return

        try:

            tree = ast.parse(
                code
            )

        except SyntaxError as error:

            self.show_error(
                error
            )

            return

        self.states = []
        self.current_step = 0
        self.running = True
        self.auto_running = False

        self.code.tag_remove(
            "current",
            "1.0",
            tk.END
        )

        self.header_status.configure(
            text="● EXECUTING",
            text_color="#38BDF8"
        )

        self.execution_type = self.analyze_code(
            tree,
            code
        )

        self.type_label.configure(
            text=self.execution_type
        )

        threading.Thread(
            target=self.execute_code,
            args=(code,),
            daemon=True
        ).start()

    # ========================================================
    # CODE ANALYZER
    # ========================================================

    def analyze_code(
        self,
        tree,
        source
    ):

        text = source.lower()

        classes = [
            node.name.lower()
            for node in ast.walk(tree)
            if isinstance(
                node,
                ast.ClassDef
            )
        ]

        if (
            "graph" in text
            or "adjacency" in text
            or "neighbors" in text
            or "edges" in text
        ):
            return "GRAPH"

        if (
            "treenode" in text
            or "bst" in text
            or "binary tree" in text
            or "root.left" in text
            or "root.right" in text
        ):
            return "TREE"

        if (
            "listnode" in text
            or ".next" in text
        ):
            return "LINKED LIST"

        if (
            "stack" in text
            and (
                ".append(" in text
                or ".pop(" in text
            )
        ):
            return "STACK"

        if (
            "queue" in text
            or "popleft" in text
            or "enqueue" in text
            or "dequeue" in text
        ):
            return "QUEUE"

        if (
            "recursion" in text
        ):
            return "RECURSION"

        # ARRAY must be checked before LOOP.
        # Array programs commonly use for-loops to traverse elements;
        # checking LOOP first incorrectly hides the array visualization.
        for node in ast.walk(tree):

            if isinstance(
                node,
                ast.List
            ):
                return "ARRAY"

        if (
            "for " in text
            or "while " in text
        ):
            return "LOOP"

        return "CODE"

    # ========================================================
    # EXECUTION ENGINE
    # ========================================================

    def execute_code(
        self,
        source
    ):

        temp = tempfile.NamedTemporaryFile(
            mode="w",
            suffix=".py",
            delete=False,
            encoding="utf-8"
        )

        temp.write(
            source
        )

        temp.close()

        filename = temp.name

        output = io.StringIO()

        captured = []

        def snapshot_tree(node, seen=None):
            if node is None:
                return None

            if seen is None:
                seen = {}

            node_id = id(node)
            if node_id in seen:
                return seen[node_id]

            copied = Node(node.value)
            seen[node_id] = copied
            copied.left = snapshot_tree(
                getattr(node, "left", None), seen
            )
            copied.right = snapshot_tree(
                getattr(node, "right", None), seen
            )
            return copied

        def trace(
            frame,
            event,
            arg
        ):

            if frame.f_code.co_filename != filename:
                return trace

            if event == "line":

                line = frame.f_lineno

                variables = {}

                for key, value in frame.f_locals.items():

                    if key.startswith("__"):
                        continue

                    try:

                        if self.safe_value(
                            value
                        ):
                            variables[key] = value

                    except:
                        pass

                # TREE FIX: root is global while Node.__init__ runs.
                if self.execution_type == "TREE":
                    tree_root = frame.f_globals.get(
                        "root",
                        frame.f_locals.get("root")
                    )

                    # User code defines its own Node class inside exec(),
                    # so isinstance(tree_root, app_Node) can be False.
                    # Detect tree nodes by their structure instead.
                    if (
                        tree_root is not None
                        and hasattr(tree_root, "value")
                        and hasattr(tree_root, "left")
                        and hasattr(tree_root, "right")
                    ):
                        variables["root"] = snapshot_tree(tree_root)

                captured.append(
                    {
                        "line": line,
                        "source": self.get_line(
                            source,
                            line
                        ),
                        "variables": variables.copy(),
                        "output": output.getvalue()
                    }
                )

            return trace

        namespace = {
            "__name__": "__main__",
            "Node": Node,
            "ListNode": ListNode,
            "GraphNode": GraphNode
        }

        try:

            sys.settrace(
                trace
            )

            with contextlib.redirect_stdout(
                output
            ):

                exec(
                    compile(
                        source,
                        filename,
                        "exec"
                    ),
                    namespace,
                    namespace
                )

        except Exception as error:

            captured.append(
                {
                    "line": 1,
                    "source": "ERROR: " + str(error),
                    "variables": {},
                    "output": output.getvalue()
                    + "\n"
                    + str(error)
                }
            )

        finally:

            sys.settrace(
                None
            )

            try:
                os.remove(
                    filename
                )
            except:
                pass

        self.after(
            0,
            lambda: self.finish_execution(
                captured
            )
        )

    # ========================================================
    # SAFE VALUE
    # ========================================================

    def safe_value(
        self,
        value
    ):

        if isinstance(
            value,
            (
                int,
                float,
                str,
                bool,
                list,
                tuple,
                dict,
                type(None)
            )
        ):
            return True

        if isinstance(
            value,
            (
                Node,
                ListNode,
                GraphNode
            )
        ):
            return True

        return False

    # ========================================================
    # FINISH
    # ========================================================

    def finish_execution(
        self,
        states
    ):

        self.states = states
        self.current_step = 0
        self.running = False

        self.header_status.configure(
            text="● COMPLETE",
            text_color="#34D399"
        )

        if states:
            self.show_state()

    # ========================================================
    # STEP
    # ========================================================

    def next(self):

        if not self.states:
            return

        if self.current_step < len(
            self.states
        ) - 1:

            self.current_step += 1
            self.show_state()

    def previous(self):

        if not self.states:
            return

        if self.current_step > 0:

            self.current_step -= 1
            self.show_state()

    # ========================================================
    # AUTO PLAY
    # ========================================================

    def auto_play(self):

        if not self.states:

            self.run()

            self.after(
                1000,
                self.start_auto
            )

            return

        self.start_auto()

    def start_auto(self):

        if not self.states:
            return

        self.auto_running = True
        self.current_step = 0

        self.auto_step()

    def auto_step(self):

        if not self.auto_running:
            return

        self.show_state()

        if self.current_step < len(
            self.states
        ) - 1:

            self.current_step += 1

            self.after(
                self.play_speed,
                self.auto_step
            )

        else:

            self.auto_running = False

            self.header_status.configure(
                text="● COMPLETE",
                text_color="#34D399"
            )

    # ========================================================
    # ADDED: SPEED CONTROL
    # ========================================================

    def change_speed(
        self,
        value
    ):

        self.play_speed = int(
            float(value)
        )

    # ========================================================
    # SHOW STATE
    # ========================================================

    def show_state(self):

        if not self.states:
            return

        state = self.states[
            self.current_step
        ]

        self.highlight_line(
            state["line"]
        )

        concept = self.detect_runtime_type(
            state
        )

        self.type_label.configure(
            text=concept
        )

        self.concept.configure(
            text=concept,
            text_color=self.get_color(
                concept
            )
        )

        self.draw_visualization(
            state,
            concept
        )

        self.draw_execution(
            state,
            concept
        )

        self.update_info(
            state,
            concept
        )

        self.progress.set(
            (
                self.current_step + 1
            )
            /
            max(
                1,
                len(self.states)
            )
        )

    # ========================================================
    # RUNTIME DETECTION
    # ========================================================

    def detect_runtime_type(
        self,
        state
    ):

        variables = state[
            "variables"
        ]

        source = state[
            "source"
        ].lower()

        declared = self.execution_type

        if declared == "TREE":
            return "TREE"

        if declared == "GRAPH":
            return "GRAPH"

        if declared == "LINKED LIST":
            return "LINKED LIST"

        if declared == "STACK":
            return "STACK"

        if declared == "QUEUE":
            return "QUEUE"

        if declared == "ARRAY":
            return "ARRAY"

        if declared == "LOOP":
            return "LOOP"

        if declared == "RECURSION":
            return "RECURSION"

        return "CODE"

    # ========================================================
    # VISUAL DISPATCHER
    # ========================================================

    def draw_visualization(
        self,
        state,
        concept
    ):

        self.canvas.delete(
            "all"
        )

        if concept == "ARRAY":
            self.draw_array(
                state
            )

        elif concept == "STACK":
            self.draw_stack(
                state
            )

        elif concept == "QUEUE":
            self.draw_queue(
                state
            )

        elif concept == "TREE":
            self.draw_tree(
                state
            )

        elif concept == "LINKED LIST":
            self.draw_linked_list(
                state
            )

        elif concept == "GRAPH":
            self.draw_graph(
                state
            )

        elif concept == "LOOP":
            self.draw_loop_main(
                state
            )

        elif concept == "RECURSION":
            self.draw_recursion(
                state
            )

        else:
            self.draw_code_state(
                state
            )

    # ========================================================
    # ARRAY
    # ========================================================

    def draw_array(
        self,
        state
    ):

        arrays = [
            (k, v)
            for k, v in state[
                "variables"
            ].items()
            if isinstance(
                v,
                list
            )
        ]

        if not arrays:
            self.draw_code_state(
                state
            )
            return

        name, arr = arrays[0]

        w = max(
            300,
            self.canvas.winfo_width()
        )

        h = max(
            250,
            self.canvas.winfo_height()
        )

        self.draw_title(
            "ARRAY",
            "#38BDF8"
        )

        if not arr:
            self.center_text(
                "ARRAY = [ ]",
                "#475569"
            )
            return

        box_w = min(
            70,
            max(
                38,
                (w - 50)
                /
                max(
                    1,
                    len(arr)
                )
                - 8
            )
        )

        box_h = min(
            52,
            max(
                35,
                h * 0.12
            )
        )

        gap = min(
            8,
            max(
                3,
                box_w * 0.08
            )
        )

        total = (
            len(arr)
            * (
                box_w + gap
            )
            - gap
        )

        start_x = (
            w - total
        ) / 2

        y = h / 2 - box_h / 2

        for i, value in enumerate(arr):

            x = (
                start_x
                +
                i
                *
                (
                    box_w + gap
                )
            )

            self.canvas.create_rectangle(
                x,
                y,
                x + box_w,
                y + box_h,
                fill="#082F49",
                outline="#38BDF8",
                width=2
            )

            self.canvas.create_text(
                x + box_w / 2,
                y + box_h / 2 - 5,
                text=str(value),
                fill="#E0F2FE",
                font=(
                    "Consolas",
                    max(
                        9,
                        int(box_w / 5)
                    ),
                    "bold"
                )
            )

            self.canvas.create_text(
                x + box_w / 2,
                y + box_h - 7,
                text=f"[{i}]",
                fill="#64748B",
                font=(
                    "Consolas",
                    8
                )
            )

    # ========================================================
    # STACK
    # ========================================================

    def draw_stack(
        self,
        state
    ):

        stack = state[
            "variables"
        ].get(
            "stack",
            []
        )

        w = max(
            300,
            self.canvas.winfo_width()
        )

        h = max(
            250,
            self.canvas.winfo_height()
        )

        self.draw_title(
            "STACK • LIFO",
            "#A78BFA"
        )

        if not isinstance(
            stack,
            list
        ):
            return

        box_w = min(
            130,
            w * 0.35
        )

        available = h - 100

        count = max(
            1,
            len(stack)
        )

        box_h = min(
            45,
            max(
                25,
                (
                    available
                    -
                    (count - 1) * 5
                )
                /
                count
            )
        )

        gap = 5

        visible = stack[
            -max(
                1,
                int(
                    available
                    /
                    (box_h + gap)
                )
            ):
        ]

        total_h = len(
            visible
        ) * (
            box_h + gap
        ) - gap

        start_y = (
            h - total_h
        ) / 2 + 15

        center = w / 2

        for i, value in enumerate(
            reversed(visible)
        ):

            y = (
                start_y
                +
                i
                *
                (
                    box_h + gap
                )
            )

            self.canvas.create_rectangle(
                center - box_w / 2,
                y,
                center + box_w / 2,
                y + box_h,
                fill="#2E1065",
                outline="#A78BFA",
                width=2
            )

            self.canvas.create_text(
                center,
                y + box_h / 2,
                text=str(value),
                fill="#EDE9FE",
                font=(
                    "Consolas",
                    11,
                    "bold"
                )
            )

        if stack:

            self.canvas.create_text(
                center + box_w / 2 + 35,
                start_y + box_h / 2,
                text="TOP",
                fill="#A78BFA",
                font=(
                    "Consolas",
                    9,
                    "bold"
                )
            )

    # ========================================================
    # QUEUE
    # ========================================================

    def draw_queue(
        self,
        state
    ):

        queue = state[
            "variables"
        ].get(
            "queue",
            []
        )

        w = max(
            300,
            self.canvas.winfo_width()
        )

        h = max(
            250,
            self.canvas.winfo_height()
        )

        self.draw_title(
            "QUEUE • FIFO",
            "#2DD4BF"
        )

        if not isinstance(
            queue,
            list
        ):
            return

        if not queue:
            self.center_text(
                "QUEUE = [ ]",
                "#475569"
            )
            return

        box_w = min(
            70,
            max(
                35,
                (
                    w - 60
                )
                /
                max(
                    1,
                    len(queue)
                )
                - 12
            )
        )

        box_h = 50
        gap = 15

        total = (
            len(queue)
            *
            (
                box_w + gap
            )
            -
            gap
        )

        start_x = (
            w - total
        ) / 2

        y = h / 2 - 20

        for i, value in enumerate(
            queue
        ):

            x = (
                start_x
                +
                i
                *
                (
                    box_w + gap
                )
            )

            self.canvas.create_rectangle(
                x,
                y,
                x + box_w,
                y + box_h,
                fill="#042F2E",
                outline="#2DD4BF",
                width=2
            )

            self.canvas.create_text(
                x + box_w / 2,
                y + box_h / 2,
                text=str(value),
                fill="#CCFBF1",
                font=(
                    "Consolas",
                    11,
                    "bold"
                )
            )

            if i < len(queue) - 1:

                self.canvas.create_line(
                    x + box_w,
                    y + box_h / 2,
                    x + box_w + gap,
                    y + box_h / 2,
                    fill="#2DD4BF",
                    width=2,
                    arrow=tk.LAST
                )

        self.canvas.create_text(
            start_x,
            y - 15,
            text="FRONT",
            fill="#2DD4BF",
            font=(
                "Consolas",
                8,
                "bold"
            )
        )

        self.canvas.create_text(
            start_x + total,
            y - 15,
            text="REAR",
            fill="#2DD4BF",
            font=(
                "Consolas",
                8,
                "bold"
            )
        )

    # ========================================================
    # TREE
    # ========================================================

    def draw_tree(
        self,
        state
    ):

        root = state[
            "variables"
        ].get(
            "root"
        )

        w = max(
            300,
            self.canvas.winfo_width()
        )

        h = max(
            250,
            self.canvas.winfo_height()
        )

        self.draw_title(
            "BINARY SEARCH TREE",
            "#22C55E"
        )

        if root is None:

            self.center_text(
                "ROOT = NULL",
                "#475569"
            )

            return

        levels = self.tree_height(
            root
        )

        top = 85

        available_h = max(
            100,
            h - top - 35
        )

        vertical_gap = min(
            72,
            available_h
            /
            max(
                1,
                levels - 1
            )
        )

        max_width = w * 0.72

        offset = min(
            125,
            max_width / 4
        )

        self.draw_tree_node(
            root,
            w / 2,
            top,
            offset,
            vertical_gap,
            w
        )

    def tree_height(
        self,
        node
    ):

        if node is None:
            return 0

        return (
            1
            +
            max(
                self.tree_height(
                    getattr(
                        node,
                        "left",
                        None
                    )
                ),
                self.tree_height(
                    getattr(
                        node,
                        "right",
                        None
                    )
                )
            )
        )

    def draw_tree_node(
        self,
        node,
        x,
        y,
        offset,
        vertical_gap,
        width
    ):

        if node is None:
            return

        radius = min(
            23,
            max(
                16,
                width / 45
            )
        )

        left = getattr(
            node,
            "left",
            None
        )

        right = getattr(
            node,
            "right",
            None
        )

        next_y = y + vertical_gap

        next_offset = max(
            30,
            offset * 0.52
        )

        if left is not None:

            left_x = x - offset

            self.canvas.create_line(
                x,
                y + radius,
                left_x,
                next_y - radius,
                fill="#22C55E",
                width=2
            )

            self.draw_tree_node(
                left,
                left_x,
                next_y,
                next_offset,
                vertical_gap,
                width
            )

        if right is not None:

            right_x = x + offset

            self.canvas.create_line(
                x,
                y + radius,
                right_x,
                next_y - radius,
                fill="#22C55E",
                width=2
            )

            self.draw_tree_node(
                right,
                right_x,
                next_y,
                next_offset,
                vertical_gap,
                width
            )

        self.canvas.create_oval(
            x - radius,
            y - radius,
            x + radius,
            y + radius,
            fill="#064E3B",
            outline="#22C55E",
            width=2
        )

        self.canvas.create_text(
            x,
            y,
            text=str(
                getattr(
                    node,
                    "value",
                    "?"
                )
            ),
            fill="#DCFCE7",
            font=(
                "Consolas",
                10,
                "bold"
            )
        )

    # ========================================================
    # LINKED LIST
    # ========================================================

    def draw_linked_list(
        self,
        state
    ):

        variables = state[
            "variables"
        ]

        head = variables.get(
            "head"
        )

        w = max(
            300,
            self.canvas.winfo_width()
        )

        h = max(
            250,
            self.canvas.winfo_height()
        )

        self.draw_title(
            "LINKED LIST",
            "#F59E0B"
        )

        if head is None:

            self.center_text(
                "HEAD = NULL",
                "#475569"
            )

            return

        nodes = []

        current = head

        visited = set()

        while (
            current is not None
            and id(current) not in visited
            and len(nodes) < 30
        ):

            visited.add(
                id(current)
            )

            nodes.append(
                current
            )

            current = getattr(
                current,
                "next",
                None
            )

        box_w = min(
            100,
            max(
                55,
                (
                    w - 80
                )
                /
                max(
                    1,
                    len(nodes)
                )
                - 15
            )
        )

        box_h = 50
        gap = 25

        total = (
            len(nodes)
            *
            (
                box_w + gap
            )
            -
            gap
        )

        start = (
            w - total
        ) / 2

        y = h / 2

        for i, node in enumerate(
            nodes
        ):

            x = (
                start
                +
                i
                *
                (
                    box_w + gap
                )
            )

            self.canvas.create_rectangle(
                x,
                y,
                x + box_w,
                y + box_h,
                fill="#451A03",
                outline="#F59E0B",
                width=2
            )

            self.canvas.create_text(
                x + box_w / 2,
                y + box_h / 2,
                text=str(
                    node.value
                ),
                fill="#FEF3C7",
                font=(
                    "Consolas",
                    11,
                    "bold"
                )
            )

            if i < len(nodes) - 1:

                self.canvas.create_line(
                    x + box_w,
                    y + box_h / 2,
                    x + box_w + gap,
                    y + box_h / 2,
                    fill="#F59E0B",
                    width=2,
                    arrow=tk.LAST
                )

        self.canvas.create_text(
            start,
            y - 18,
            text="HEAD",
            fill="#F59E0B",
            font=(
                "Consolas",
                8,
                "bold"
            )
        )

    # ========================================================
    # GRAPH
    # ========================================================

    def draw_graph(
        self,
        state
    ):

        variables = state[
            "variables"
        ]

        nodes = []

        for name, value in variables.items():

            if isinstance(
                value,
                GraphNode
            ):
                nodes.append(
                    value
                )

        w = max(
            300,
            self.canvas.winfo_width()
        )

        h = max(
            250,
            self.canvas.winfo_height()
        )

        self.draw_title(
            "GRAPH",
            "#F472B6"
        )

        if not nodes:

            self.center_text(
                "GRAPH READY",
                "#475569"
            )

            return

        nodes = nodes[
            :12
        ]

        cx = w / 2
        cy = h / 2 + 10

        radius_layout = min(
            w,
            h
        ) * 0.30

        positions = {}

        for i, node in enumerate(
            nodes
        ):

            angle = (
                2
                *
                math.pi
                *
                i
                /
                len(nodes)
            )

            positions[
                id(node)
            ] = (
                cx
                +
                radius_layout
                *
                math.cos(angle),
                cy
                +
                radius_layout
                *
                math.sin(angle)
            )

        drawn_edges = set()

        for node in nodes:

            x1, y1 = positions[
                id(node)
            ]

            for neighbor in getattr(
                node,
                "neighbors",
                []
            ):

                if id(neighbor) not in positions:
                    continue

                x2, y2 = positions[
                    id(neighbor)
                ]

                edge = tuple(
                    sorted(
                        [
                            id(node),
                            id(neighbor)
                        ]
                    )
                )

                if edge in drawn_edges:
                    continue

                drawn_edges.add(
                    edge
                )

                self.canvas.create_line(
                    x1,
                    y1,
                    x2,
                    y2,
                    fill="#EC4899",
                    width=2
                )

        node_r = min(
            24,
            max(
                17,
                w / 40
            )
        )

        for node in nodes:

            x, y = positions[
                id(node)
            ]

            self.canvas.create_oval(
                x - node_r,
                y - node_r,
                x + node_r,
                y + node_r,
                fill="#500724",
                outline="#F472B6",
                width=2
            )

            self.canvas.create_text(
                x,
                y,
                text=str(
                    node.value
                ),
                fill="#FCE7F3",
                font=(
                    "Consolas",
                    10,
                    "bold"
                )
            )

    # ========================================================
    # LOOP
    # ========================================================

    def draw_loop_main(
        self,
        state
    ):

        w = max(
            300,
            self.canvas.winfo_width()
        )

        h = max(
            250,
            self.canvas.winfo_height()
        )

        self.draw_title(
            "LOOP EXECUTION",
            "#F472B6"
        )

        variables = state[
            "variables"
        ]

        value = None

        for key in (
            "i",
            "j",
            "k",
            "value"
        ):

            if key in variables:
                value = variables[
                    key
                ]
                break

        cx = w / 2

        self.canvas.create_rectangle(
            cx - 110,
            h * 0.30,
            cx + 110,
            h * 0.48,
            fill="#2D0A3D",
            outline="#F472B6",
            width=2
        )

        self.canvas.create_text(
            cx,
            h * 0.39,
            text="ITERATION",
            fill="#F9A8D4",
            font=(
                "Consolas",
                11,
                "bold"
            )
        )

        self.canvas.create_text(
            cx,
            h * 0.58,
            text=f"CURRENT = {value}",
            fill="#F472B6",
            font=(
                "Consolas",
                15,
                "bold"
            )
        )

    # ========================================================
    # RECURSION
    # ========================================================

    def draw_recursion(
        self,
        state
    ):

        w = max(
            300,
            self.canvas.winfo_width()
        )

        h = max(
            250,
            self.canvas.winfo_height()
        )

        self.draw_title(
            "RECURSION • CALL STACK",
            "#F59E0B"
        )

        self.canvas.create_rectangle(
            w * 0.35,
            h * 0.25,
            w * 0.65,
            h * 0.75,
            fill="#451A03",
            outline="#F59E0B",
            width=2
        )

        self.canvas.create_text(
            w / 2,
            h / 2,
            text="CALL\nSTACK",
            fill="#FEF3C7",
            font=(
                "Consolas",
                17,
                "bold"
            )
        )

    # ========================================================
    # CODE
    # ========================================================

    def draw_code_state(
        self,
        state
    ):

        self.center_text(
            "EXECUTING CODE",
            "#475569"
        )

    # ========================================================
    # EXECUTION INFO
    # ========================================================

    def draw_execution(
        self,
        state,
        concept
    ):

        self.loop_canvas.delete(
            "all"
        )

        self.loop_canvas.create_text(
            150,
            25,
            text="EXECUTION FLOW",
            fill="#64748B",
            font=(
                "Consolas",
                9,
                "bold"
            )
        )

        self.loop_canvas.create_rectangle(
            35,
            50,
            265,
            95,
            fill="#0F172A",
            outline=self.get_color(
                concept
            ),
            width=2
        )

        self.loop_canvas.create_text(
            150,
            72,
            text=f"STEP {self.current_step + 1}",
            fill="#E2E8F0",
            font=(
                "Consolas",
                11,
                "bold"
            )
        )

        self.loop_canvas.create_line(
            150,
            95,
            150,
            130,
            fill=self.get_color(
                concept
            ),
            width=2,
            arrow=tk.LAST
        )

        self.loop_canvas.create_rectangle(
            55,
            135,
            245,
            170,
            fill="#0F172A",
            outline="#334155",
            width=1
        )

        self.loop_canvas.create_text(
            150,
            152,
            text="STATE UPDATED",
            fill="#94A3B8",
            font=(
                "Consolas",
                8,
                "bold"
            )
        )

    # ========================================================
    # INFO
    # ========================================================

    def update_info(
        self,
        state,
        concept
    ):

        complexity = self.calculate_complexity()

        text = (
            f"CONCEPT     : {concept}\n"
            f"LINE        : {state['line']}\n"
            f"STEP        : {self.current_step + 1}/"
            f"{len(self.states)}\n\n"
            f"CURRENT CODE\n"
            f"{state['source']}\n\n"
            f"VARIABLES\n"
            f"{state['variables']}\n\n"
            f"OUTPUT\n"
            f"{state['output']}\n\n"
            f"COMPLEXITY\n"
            f"TIME  : {complexity[0]}\n"
            f"SPACE : {complexity[1]}"
        )

        self.info.configure(
            state="normal"
        )

        self.info.delete(
            "1.0",
            tk.END
        )

        self.info.insert(
            "1.0",
            text
        )

        self.info.configure(
            state="disabled"
        )

    # ========================================================
    # COMPLEXITY
    # ========================================================

    def calculate_complexity(
        self
    ):

        source = self.code.get(
            "1.0",
            "end-1c"
        )

        try:

            tree = ast.parse(
                source
            )

        except:

            return (
                "Unknown",
                "Unknown"
            )

        loops = 0
        nested = 0
        functions = 0

        for node in ast.walk(
            tree
        ):

            if isinstance(
                node,
                (
                    ast.For,
                    ast.While
                )
            ):
                loops += 1

            if isinstance(
                node,
                ast.FunctionDef
            ):
                functions += 1

        max_depth = self.loop_depth(
            tree
        )

        if max_depth >= 3:
            time_complexity = "O(n³)"
        elif max_depth == 2:
            time_complexity = "O(n²)"
        elif max_depth == 1:
            time_complexity = "O(n)"
        elif functions > 0:
            time_complexity = "Depends on recursion"
        else:
            time_complexity = "O(1)"

        list_count = source.count(
            "["
        )

        if functions > 0:
            space_complexity = "O(n) / call stack"
        elif list_count > 0:
            space_complexity = "O(n)"
        else:
            space_complexity = "O(1)"

        return (
            time_complexity,
            space_complexity
        )

    def loop_depth(
        self,
        tree,
        depth=0
    ):

        best = depth

        for node in ast.iter_child_nodes(
            tree
        ):

            if isinstance(
                node,
                (
                    ast.For,
                    ast.While
                )
            ):

                best = max(
                    best,
                    self.loop_depth(
                        node,
                        depth + 1
                    )
                )

            else:

                best = max(
                    best,
                    self.loop_depth(
                        node,
                        depth
                    )
                )

        return best

    # ========================================================
    # HELPERS
    # ========================================================

    def draw_title(
        self,
        text,
        color
    ):

        w = self.canvas.winfo_width()

        self.canvas.create_text(
            w / 2,
            30,
            text=text,
            fill=color,
            font=(
                "Consolas",
                16,
                "bold"
            )
        )

    def center_text(
        self,
        text,
        color
    ):

        w = self.canvas.winfo_width()
        h = self.canvas.winfo_height()

        self.canvas.create_text(
            w / 2,
            h / 2,
            text=text,
            fill=color,
            font=(
                "Consolas",
                13,
                "bold"
            )
        )

    def get_color(
        self,
        concept
    ):

        return {
            "ARRAY": "#38BDF8",
            "STACK": "#A78BFA",
            "QUEUE": "#2DD4BF",
            "TREE": "#22C55E",
            "GRAPH": "#F472B6",
            "LINKED LIST": "#F59E0B",
            "LOOP": "#F472B6",
            "RECURSION": "#F59E0B",
            "CODE": "#64748B"
        }.get(
            concept,
            "#64748B"
        )

    def get_line(
        self,
        source,
        line
    ):

        lines = source.splitlines()

        if (
            0 < line <= len(lines)
        ):
            return lines[
                line - 1
            ].strip()

        return ""

    def highlight_line(
        self,
        line
    ):

        self.code.tag_remove(
            "current",
            "1.0",
            tk.END
        )

        self.code.tag_add(
            "current",
            f"{line}.0",
            f"{line}.end"
        )

        self.code.see(
            f"{line}.0"
        )

    def show_error(
        self,
        error
    ):

        self.header_status.configure(
            text="● SYNTAX ERROR",
            text_color="#FB7185"
        )

        self.canvas.delete(
            "all"
        )

        self.center_text(
            "SYNTAX ERROR",
            "#FB7185"
        )

    def draw_idle(
        self
    ):

        self.canvas.delete(
            "all"
        )

        self.center_text(
            "READY FOR EXECUTION",
            "#334155"
        )

    def refresh_visual(
        self
    ):

        if self.states:

            self.show_state()

    def reset(
        self
    ):

        self.auto_running = False
        self.running = False

        self.states = []
        self.current_step = 0

        self.canvas.delete(
            "all"
        )

        self.loop_canvas.delete(
            "all"
        )

        self.code.tag_remove(
            "current",
            "1.0",
            tk.END
        )

        self.progress.set(
            0
        )

        self.concept.configure(
            text="WAITING",
            text_color="#64748B"
        )

        self.type_label.configure(
            text="CODE"
        )

        self.header_status.configure(
            text="● READY",
            text_color="#34D399"
        )

        self.info.configure(
            state="normal"
        )

        self.info.delete(
            "1.0",
            tk.END
        )

        self.info.configure(
            state="disabled"
        )

        self.draw_idle()


# ============================================================
# START
# ============================================================

if __name__ == "__main__":

    app = CodeFlow()

    app.mainloop()
